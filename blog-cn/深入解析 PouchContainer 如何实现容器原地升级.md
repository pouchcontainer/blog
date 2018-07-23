# 背景

阿里巴巴集团内部，容器使用方式有很大一部分是富容器模式，像这种基于传统虚拟机运维模式下的富容器，其中也有一定数量容器仍然是有状态的。有状态服务的更新和升级是企业内部频率很高的一个日常操作，对于以镜像为交付的容器技术来说，服务的更新和升级，对应的容器操作实际上是两步：旧镜像容器的删除，以及新镜像容器的创建。而有状态服务的升级，则要求保证新容器必须继承旧容器所有的资源，比如网络、存储等信息。下面给出两个实际的业务案例来直观阐述富容器业务发布场景需求：

* 客户案例一：某数据库业务，在第一次创建容器服务时，会将远程的数据下载到本地，作为数据库的初始数据。因为数据库初始化过程会比较长，所以在之后可能存在的服务升级过程中，新容器需要继承旧容器的存储数据，来降低业务发布的时间；
* 客户案例二：某中间件服务，业务采取服务注册的模式，即所有新扩容的容器 IP 必须首先注册到服务器列表中，否则新扩容业务容器不可用。在业务容器每次升级发布时，需要保证新容器继承旧容器 IP，否则会导致新发布的服务不可用。

现在很多企业都是使用 Moby 作为容器引擎，但 Moby 的所有 API 中并没有一个接口来对标容器升级这一操作。而组合 API 的方式，必然会增加很多 API 请求次数，比如需要请求容器的增删 API，需要请求 IP 保留的 API 等等，还可能增加升级操作失败的风险。

基于以上背景，PouchContainer 在容器引擎层面提供了一个 `upgrade` 接口，用于实现容器的原地升级功能。将容器升级功能下沉到容器引擎这一层来做，对于操作容器相关资源更加方便，并且减少很多 API 请求，让容器升级操作变得更加高效。

# Upgrade 功能具体实现

## 容器底层存储介绍

PouchContainer 底层对接的是 Containerd v1.0.3 ，对比 Moby，在容器存储架构上有很大的差别，所以在介绍 PouchContainer 如何实现容器原地升级功能之前，有必要先简单介绍一下在 PouchContainer  中一个容器的存储架构：


![image.png | center | 600x336.3525091799266](https://cdn.yuque.com/lark/0/2018/png/95961/1527735535637-5afc58e6-31ef-400c-984c-a9d7158fd40d.png "")


对比 Moby 中容器存储架构，PouchContainer 主要不一样的地方：
* PouchContainer  中没有了 GraphDriver 和 Layer 的概念，新的存储架构里引入了 Snapshotter 和 Snapshot，从而更加拥抱 CNCF 项目 containerd 的架构设计。Snapshotter 可以理解为存储驱动，比如 overlay、devicemapper、btrfs 等。Snapshot 为镜像快照，分为两种：一种只读的，即容器镜像的每一层只读数据；一种为可读写的，即容器可读写层，所有容器增量数据都会存储在可读写 Snapshot 中；
* Containerd 中容器和镜像元数据都存储在 boltdb 中，这样的好处是每次服务重启不需要通过读取宿主机文件目录信息来初始化容器和镜像数据，而是只需要初始化 boltdb。

## Upgrade 功能需求

每一个系统和功能设计之初，都需要详细调研该系统或功能需要为用户解决什么疼点。经过调研阿里内部使用容器原地升级功能的具体业务场景，我们对 `upgrade` 功能设计总结了三点要求：
* 数据一致性
* 灵活性
* 鲁棒性

数据一致性指 `upgrade` 前后需要保证一些数据不变：
* 网络：升级前后，容器网络配置要保持不变；
* 存储：新容器需要继承旧容器的所有 volume ；
* Config：新容器需要继承旧容器的某一些配置信息，比如 Env, Labels 等信息；

灵活性指 `upgrade` 操作在旧容器的基础上，允许引入新的配置：
* 允许修改新容器的 cpu、memory 等信息；
* 对新的镜像，即要支持指定新的 `Entrypoint` ，也要允许继承旧容器的 `Entrypoint` ；
* 支持给容器增加新的 volume，新的镜像中可能会包含新的 volume 信息，在新建容器时，需要对这部分 volume 信息进行解析，并创建新的 volume。  

鲁棒性是指在进行容器原地升级操作期间，需要对可能出现的异常情况进行处理，支持回滚策略，升级失败可以回滚到旧容器。

## Upgrade 功能具体实现
### Upgrade API 定义

首先说明一下 `upgrade`  API 入口层定义，用于定义升级操作可以对容器的哪些参数进行修改。如下 `ContainerUpgradeConfig` 的定义，容器升级操作可以对容器 `ContainerConfig` 和 `HostConfig` 都可以进行操作，如果在 PouchContainer github 代码仓库的 `apis/types` 目录下参看这两个参数的定义，可以发现实际上，`upgrade` 操作可以修改旧容器的 __所有__ 相关配置。 
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

### Upgrade 详细操作流程

容器 `upgrade` 操作，实际上是在保证网络配置和原始 volume 配置不变的前提下，进行旧容器的删除操作，以及使用新镜像创建新容器的过程，如下给出了 `upgrade` 操作流程的详细说明：
* 首先需要备份原有容器的所有操作，用于升级失败之后，进行回滚操作；
* 更新容器配置参数，将请求参数中新的配置参数合并到旧的容器参数中，使新配置生效；
* 镜像 `Entrypoint` 参数特殊处理：如果新的参数中指定了 `Entrypoint` 参数，则使用新的参数；否则查看旧容器的 `Entrypoint` ，如果该参数是通过配置参数指定，而不是旧镜像中自带的，则使用旧容器的 `Entrypoint` 作为新容器的 `Entrypoint` ；如果都不是，最后使用新镜像中的 `Entrypoint` 最为新创建容器的 `Entrypoint` 。对新容器 `Entrypoint` 这样处理的原因是为了保持容器服务入口参数的连续性。
* 判断容器的状态，如果是 running 状态的容器，首先 stop 容器；之后基于新的镜像创建一个新的 Snapshot 作为新容器读写层；
* 新的 Snapshot 创建成功之后，再次判断旧容器升级之前的状态，如果是 running 状态，则需要启动新的容器，否则不需要做任何操作；
* 最后进行容器升级清理工作，删掉旧的 Snapshot，并将最新配置进行存盘。

### Upgrade 操作回滚

`upgrade` 操作可能会出现一些异常情况，现在的升级策略是在出现异常情况时，会进行回滚操作，恢复到原来旧容器的状态，在这里我们需要首先定义一下 __升级失败情况__ :
* 给新容器创建新的资源时失败，需要执行回滚操作：当给新容器创建新的 Snapshot，Volumes 等资源时，会执行回滚操作；
* 启动新容器出现系统错误时，需要执行回滚操作：即在调用 containerd API 创建新的容器时如果失败则会执行回滚操作。如果 API 返回正常，但容器内的程序运行异常导致容器退出的情况，不会执行回滚操作。
如下给出了回滚操作的一个基本操作：
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

在升级过程中，如果出现异常情况，会将新创建的 Snapshot 等相关资源进行清理操作，在回滚阶段，只需要恢复旧容器的配置，然后用恢复后的配置文件启动一个新容器即可。

### Upgrade 功能演示

* 使用 `ubuntu` 镜像创建一个新容器：
```bash
$ pouch run --name test -d -t registry.hub.docker.com/library/ubuntu:14.04 top
43b75002b9a20264907441e0fe7d66030fb9acedaa9aa0fef839ccab1f9b7a8f

$ pouch ps
Name   ID       Status         Created          Image                                            Runtime
test   43b750   Up 3 seconds   3 seconds ago   registry.hub.docker.com/library/ubuntu:14.04   runc
```

* 将 `test` 容器的镜像升级为 `busybox` :
```bash
$ pouch upgrade --name test registry.hub.docker.com/library/busybox:latest top
test
$ pouch ps
Name   ID       Status         Created          Image                                            Runtime
test   43b750   Up 3 seconds   34 seconds ago   registry.hub.docker.com/library/busybox:latest   runc
```

如上功能演示，通过 `upgrade` 接口，直接将容器的镜像替换为新的镜像，而其他配置都没有变化。

# 总结

在企业生产环境中，容器 `upgrade` 操作和容器扩容、缩容操作一样也是的一个高频操作，但是，不管是在现在的 Moby 社区，还是 Containerd 社区都没有一个与该操作对标的 API，PouchContainer 率先实现了这个功能，解决了容器技术在企业环境中有状态服务更新发布的一个痛点问题。PouchContainer 现在也在尝试与其下游依赖组件服务如 Containerd 保持紧密的联系，所以后续也会将 `upgrade` 功能回馈给 Containerd 社区，增加 Containerd 的功能丰富度。 
