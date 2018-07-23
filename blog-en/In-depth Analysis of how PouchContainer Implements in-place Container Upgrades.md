## Background
"Rich container" mode is mainly applied in Alibaba, which is based on the traditional virtual machine operation and maintenance mode. A certain number of such containers are still stateful. It is frequent to update the stateful service in daily development. In terms of image-based container technology, there are two steps for updating services: the remove of the old image, and the create of the new image. Moreover, in order to update the stateful services, all of the resources including network, storage, etc., should be inherited. The following two business cases can demonstrate the requirement of rich container in the scenario of business release.

* Case 1: In the case of a database business, the data from remote server would be downloaded to the local machine as the initial data at the first time the container service was created. Due to the long time that the process of initiating database takes, the new container should inherit data from the old one in the process of service upgrade that may exist in the future to decrease the waiting time of releasing business.
* Case 2: In the case of a middleware service, the business has "Service Registration" mode, which means all containers that extends their volumes should be registered to the list of servers. Otherwise, these new containers cannot work. Therefore, every time the containers get updated, the new containers should inherit the IP address from the old containers, or the released services cannot work.

Nowadays, Moby is popular as the engine of the container. However, there is no even one API can be used to update the containers. Even though updating containers could be done by calling multiple APIs, this would definitely increase the number of requests to extra APIs, like the API to add or delete containers and the API to reserve IP address. Moreover, this could increase the risk of failure of the upgrade.

Based on the above background, PouchContainer provides an API `upgrade` on the engine level to realize the in-place updating functionality. It is more convenient and efficient to realize the updating functionality on the engine level due to the less API requests.

## Implementation of functionality upgrade
### Introduction to the underlying storage of containers
PouchContainer relies on Containerd v1.0.3, which is different from Moby in terms of storage architecture. Therefore, it is necessary to introduce the storage architecture in PouchContainer before introducing its mechanism of in-place upgrading functionality.

![image.png | center | 600x336.3525091799266](https://cdn.yuque.com/lark/0/2018/png/95961/1527735535637-5afc58e6-31ef-400c-984c-a9d7158fd40d.png "")

Compared with the storage architecture in Moby, the main unique features of PouchContainer are:
* There is no concept of GraphDriver or Layer in PouchContainer, on the contrast, PouchContainer introduces Snapshotter and Snapshot, which embraces the architecture design of containerd of project CNCF. Snapshooter can be regarded as the storage driver, like overlay, devicemapper, btrfs, etc. Snapshot is the snapshot of the image, which has two types: the read-only version which is the read-only data in every layer of the container; the read-write version which is the read-write layer of the container that saves all the incremental data of the container.
* The container and the unit data of the image in Containerd are stored in boltdb, which brings an advantage that initiating contianer and image data can be done by initiating boltdb instead of reading host file directory information.

## The requirement of functionality upgrade
At the early stage of the design of any system or functionality, the thorough research should be done to discover the pain points of users that we need to solve by the designed system or functionality. Three requirements were concluded after the research about the business scenarios where the in-place upgrade of the container can be used during the daily development in Alibaba, which are:
* Data Coherency
* Flexibility
* Robustness

Data Coherency refers to certain data remaining unchanged before and after the execution of `upgrade`:
* Network: the configuration of the network remains the same before and after the upgrade;
* Storage: new container should inherit all the volume from the old container;
* Config: new container should inherit some config information from the old container, e.g., `Env`, `Labels`, etc.

Flexibility means that the new config is able to be introduced while the `upgrade` is operated on old container:
* The information of new container like cpu and memory can be modified;
* With regard to new image, except that the `Entrypoint` of the old container can be inherited, the new `Entrypoint` can be supported as well;
* New volume can be added into the container, and new image could include the information of the new volume. When creating a new container, this part of the volume information should be parsed and a new volume is created.

Robustness means that rollback strategy is supported during the in-place upgrade of the container to deal with the exceptions. If the upgrade was failed, the container would be rollbacked to the old version.

## Detailed implementation of upgrade
### Definition of upgrade API
First off, the definition of the entry layer of the upgrade API  defines which parameters of the container can be modified by the upgrade operation. As shown in the following ContainerUpgradeConfig, the container upgrade operation can operate on both ContainerConfig and HostConfig. If you refer to the definition of these two parameters in the apis/types directory of the PouchContainer github repository, you can find that the upgrade operation can modify __All__ related configurations of the old container.
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

## Detailed workflow of upgrade
The `upgrade` operation of the container is actually deleting the old container and creating a new one using the new image with the same configuration of the network and initial volume. The detailed workflow of `upgrade` is divided into the following steps:
* First off, all the operations of the initial container should be copied, which would be used to rollback in case of failed upgrade;
* Updating configuration of new container. The new configuration in the request parameter would be merged with the configuration of the old container, and the new configuration would work.
* Parameter `Entrypoint` of image would be processed specially: if parameter `Entrypoint` was assigned in the new request parameter, this `Entrypoint` would be used; otherwise `Entrypoint` of the old container would be checked. If it was assigned from configuration instead of image, `Entrypoint` of the old container would be used as `Entrypoint` of the new container. If none of these conditions was matched, `Entrypoint` from new image should be used as `Entrypoint` of the new container. The reason why such logic was invoked to process `Entrypoint` is to keep its continuity;
* Then, the status of the container would be checked. If the status was "running", the container would be stopped; after that, a new `Snapshot` was created based on the new image and was used as the read-write layer of the new container;
* The status of the old container before upgrading would be checked again after the new Snapshot was created. If it was "running", the new container should be started, otherwise nothing needs to be done;
* Finally, the old Snapshot was deleted, and the latest configuration was saved.

## The rollback of upgrade
Some exceptions could appear when using `upgrade`, and the current upgrade strategy is to rollback to the old container status in the case of exceptions. Here we need to first define the situation of the upgrade failure:
* Failed to create a new resource for the new container, the rollback operation need to be performed: the rollback operation will be performed when the new Snapshot, Volumes, etc. resources are created for the new container;
* The rollback operation will be performed if the system errors happen when starting a new container: the rollback operation will be performed if failed to create the new container when calling containerd API. The rollback operation will not be performed if the container is exited caused by the abnormal program inside the container, even if the response of API is success. A basic operation of the rollback operation is given as follows:
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
                        logrus.Errorf("failed to mark container %s stop status: %s", c.ID(), err.Error()
                }
        }
}()
```

During the upgrade process, the newly created Snapshot and other related resources will be cleaned up if an abnormal situation occurs. In the rollback phase, only the configuration of the old container needs to be restored, and then a new container can be started by using the restored configuration file.

## Demo of upgrade
* Using `ubuntu` image to create a new container
```bash
$ pouch run --name test -d -t registry.hub.docker.com/library/ubuntu:14.04 top
43b75002b9a20264907441e0fe7d66030fb9acedaa9aa0fef839ccab1f9b7a8f

$ pouch ps
Name   ID       Status         Created          Image                                            Runtime
test   43b750   Up 3 seconds   3 seconds ago   registry.hub.docker.com/library/ubuntu:14.04   runc
```
* Upgrading the image of container `test` to `busybox`:
```bash
$ pouch upgrade --name test registry.hub.docker.com/library/busybox:latest top
test
$ pouch ps
Name   ID       Status         Created          Image                                            Runtime
test   43b750   Up 3 seconds   34 seconds ago   registry.hub.docker.com/library/busybox:latest   runc
```

As shown in the above demo, the image of the container is directly replaced with the new image through the `upgrade` interface, and the other configurations are unchanged.

## Conclude
In the enterprise production environment, the `upgrade` operation of the container is also a frequent operation similar to the container expansion and shrinking operations. However, neither the current Moby community nor the Containerd community has an API similar to `upgrade` that PouchContainer has. PouchContainer is the first one to implement this functionality, which solves a painful problem of container technology about updating and releasing stateful services in the enterprise environment. Currently, PouchContainer is also trying to maintain close contact with its downstream dependent component services such as Containerd, so the `upgrade` functionality will be fed back to the Containerd community to increase the functionality richness of Containerd.

---
origin doc: https://github.com/pouchcontainer/blog/blob/master/blog-cn/%E6%B7%B1%E5%BA%A6%E8%A7%A3%E6%9E%90%20PouchContainer%20%E7%9A%84%E5%AF%8C%E5%AE%B9%E5%99%A8%E6%8A%80%E6%9C%AF.md
