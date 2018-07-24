# Background

A considerable portion of containers used in Alibaba Inc are rich containers. Among these rich containers that are operated and maintained on traditional virtual machine, some still keep information of states. For the cooperation, updating and upgrading the stateful services are more than frequent. As for a container technology that delivers by images, updating and upgrading the service requires two steps: deleting the old image container and creating a new container. However, a stateful upgrade must guarantee that new container inherits all resources from the old container, such as internet, storage and other information. Here are two examples of rich containers used in business to illustrate the necessary conditions of deploying rich container services:

* Case One: Some database service requires to download remote data to local machine as initial data of the database. Database initialization can potentially take up a lot of time. Therefore, for potential service upgrade s in the future, new container has to inherit storage data from the old container to optimize the time needed to deploy the service;
* Case Two: Some middleware service follows service registry pattern, which means any containers that are resized have to register their IPs with the service registry, otherwise the container becomes unavailable. It is also crucial to guarantee that new container inherits IP from the old container when container have new releases, otherwise the most recently deployed service becomes unavailable.

Nowadays, cooperations use Moby as their container engine but there is no interface at all in Moby’s API that  directly supports container upgrading. Combining API will unavoidably increase API requests, for example, API request to add or delete, to keep… By using combined API, there is also increasing risk of failures. 

Based on the above background, PouchContainer provides an `upgrade` interface in the container engine level to achieve update-in-place. By moving the upgrading function to engine level, manipulating container related resources becomes more convenient and more efficient due to decreasing API requests. 

# Upgrade Implementation Details 

## Introduction of Container’s Underlying Storage

PouchContainer connects with Containerd v1.0.3 in base layer. Compared with Moby, PouchContainer is quite different in the container storage architecture. Therefore, before manifesting how PouchContainer implements update-in-place, it is necessary to give an introduction regarding the storage architecture of PouchContainer:


![image.png | center | 600x336.3525091799266](https://cdn.yuque.com/lark/0/2018/png/95961/1527735535637-5afc58e6-31ef-400c-984c-a9d7158fd40d.png "")


Comparing with the container storage architecture of Moby, there are the main differences in PouchCaintainer:
* PouchContainer removes the concepts of GraphDriver and Layer and introduces Snapshotter and Snapshot to the new storage architecture. As a result, PouchContainer embraces more the architectural design of containerd in CNCF project. Snapshotter can be understood as a storage driver, such as overlays, devicemapper, BTRFS, etc. Snapshot has two types: One is read-only, which is the read-only data at each layer of the container image; the other one is read/write, which is a type of layer in container where all incremental data of container is stored in.
* Metadata of both container and image in Containerd are stored in boltdb. Such design only needs to initialize boltdb instead of reading the host file directory to initialize data of container and image.

## Upgrade Requirements

At the early stage of designing each system and function, it is necessary to investigate what pain points the system or feature solves for users. After researching the specific situations in which developers in Alibaba use update-in-place of container, there are three requirements for `upgrade` design:
* Data consistency
* Flexibility
* Robustness

Data consistency means that some data need to remain consistent after `upgrade`:
* Network: network configuration of container should remain consistent after upgrade;
* Storage: new container needs to inherit all volume of the old one;
* Config: new container needs to inherit some of configuration of the old one, such as Env, Labels, etc;

Flexibility means that developers are allowed to add new configuration when implementing `upgrade`:
* Allowed to modify the CPU, memory, and other information of new container;
* Supported to specify a new `Entrypoint` as well as inherit the old container's `Entrypoint` for a new image;
* Supported for adding new volume to container. New information of volume may be included in the new image, which needs to be parsed.  

Robustness refers to handling potential exception when implementing update-in-place in container, and supporting roll-back of the old container to deal with exceptions occurred during upgrading.

## Upgrade Implementation Details
### Upgrade API Definition 

To define how `upgrade`  can modify parameters of container, it is important to first explain upgrade’s API. As the definition of `ContainerUpgradeConfig` has shown, it can upgrade both `ContainerConfig` and `HostConfig`. In fact, according to the definitions of these parameters under `apis/types` of PouchContainer github project, `upgrade` modifies __all__ configurations regarding the old container. 
```go
// ContainerUpgradeConfig ContainerUpgradeConfig is used for API "POST /containers/upgrade".
// It wraps all kinds of config used in container upgrade.
// It can be used to encode client params in client and unmarshal request body in daemon side.
//
// swagger:model ContainerUpgradeConfig

type ContainerUpgradeConfig struct {
	ContainerConfig

	// host config
	HostConfig *HostConfig `json:"HostConfig,omitempty"`
}
```

### Upgrade Instructions

Container `upgrade` is in fact deleting the old container and creating a new container from a new image while maintaining the internet configuration and original volume configuration. Here are more detailed instruction of `upgrade`: 
* At first, log all transactions of the original container to make roll-back possible after failure.
* Update container configuration parameters by adding new configuration parameters in the request to container to activate new configuration.
* Special treatment of `Entrypoint` in the image: if new params have assigned value to `Entrypoint` then use the new value. Otherwise, query `Entrypoint` of the old container and if it is assigned via configuration parameter not the old image, then new container inherit `Entrypoint` from old container. If neither is the case, use `Entrypoint` in the new image as new container’s `Entrypoint`. The reason of doing these is to keep the consistency of  input params. 
* Decide the state of container. If state is Running, stop container at first and then create a new Snapshot, based on new image, as read/write layer of the new container. 
* After creating new Snapshot, re-decide the state of old container. If state is Running, then start new container, otherwise do nothing.
* At last clean up after container upgrade, delete old Snapshot and persist the latest configuration to disk.

### Upgrade Roll-back

`upgrade` may raise a few exceptions and now the policy is to roll back when exceptions occurred and to change the container back to its original state. Here we will further explain __upgrade failure situations__:
* When creating resources for new container fails, roll-back has to be executed: when making resources such as Snapshot, Volumes for new container, roll-back will be executed;
* When errors occurred during startup of new container, roll-back has to be executed: in other words, calling Containerd API fails when creating new container, roll-back will be executed. If API returns success while applications running inside container errors and shuts down the container, roll-back will not happen. Here is an example for roll-back:
```go
defer func() {
	if !needRollback {
		return
	}

	// rollback to old container.
	c.meta = &backupContainerMeta

	// create a new containerd container.
	if err := mgr.createContainerdContainer(ctx, c); err != nil {
		logrus.Errorf("failed to rollback upgrade action: %s", err.Error())
		if err := mgr.markStoppedAndRelease(c, nil); err != nil {
			logrus.Errorf("failed to mark container %s stop status: %s", c.ID(), err.Error())
		}
	}
}()
```

When upgrading, if exceptions are raised, recently initialized resources such as Snapshot will be garbage collected and therefore, in roll-back phase, only things need to be done are reverting configuration to its old state and start a new container with reverted configuration. 

### Upgrade Demonstration

* Using `ubuntu` image to create a new container:
```bash
$ pouch run --name test -d -t registry.hub.docker.com/library/ubuntu:14.04 top
43b75002b9a20264907441e0fe7d66030fb9acedaa9aa0fef839ccab1f9b7a8f

$ pouch ps
Name   ID       Status         Created          Image                                            Runtime
test   43b750   Up 3 seconds   3 seconds ago   registry.hub.docker.com/library/ubuntu:14.04   runc
```

* Upgrading the image of test container to `busybox` : 
```bash
$ pouch upgrade --name test registry.hub.docker.com/library/busybox:latest top
test
$ pouch ps
Name   ID       Status         Created          Image                                            Runtime
test   43b750   Up 3 seconds   34 seconds ago   registry.hub.docker.com/library/busybox:latest   runc
```

As shown in the demo, image of the container is directly replaced with new image through `upgrade` interface, and the other configurations remain unchanged.

# Summary

In production environment of enterprise, `upgrade` is one of the frequent operations in container. However, whether in Moby community  nor in Containerd community, there is no corresponding API of upgrade. PouchContainer is the first to solve the pain point of stateful service update in the enterprise environment of container technology. PouchContainer is also trying to keep close contact with services from dependent downstream component such as Containerd, and to provide feedback of `upgrade` to the Containerd community later, in order to diversify Containerd.
