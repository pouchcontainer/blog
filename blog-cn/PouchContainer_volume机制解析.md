PouchContainer volume是专门用来解决容器的数据持久化的机制，想要了解volume的机制，就需要了解PouchContainer的镜像机制。PouchContainer，和Docker一样，实现了镜像的分层机制。所谓镜像分层机制，是指容器的镜像实际上是由多个只读的镜像层(layer)叠加而成，这样不同的镜像就可以复用镜像层，大大加快了镜像分发的效率，同时也减少了容器启动时间。当容器需要启动时，pouchd（下文中提到的pouchd均指PouchContainer daemon）会在启动镜像的最上层添加一个读写层，后续容器所有的读写操作就会记录在这个读写层中。这样也引入了一个问题，那就是容器数据的持久化。假如我们将容器删除，再次通过该镜像启动时，容器之前所做的修改都丢失了，这对于有状态的应用(如数据库)是致命的。

volume绕过了镜像机制，让容器中的数据以正常的文件或者目录的形式存在于宿主机上，当容器停止或删除时，并不会影响到volume中的数据，从而实现了数据的持久化，而且volume数据可以在不同的container之间共享。

## 1. PouchContainer volume整体架构

该部分内容可能涉及PouchContainer的volume源码实现。

PouchContainer volume整体架构目前主要由以下几部分构成：

* __VolumeManager__：该结构是volume相关操作的入口。
* __Core__：Core是volume的核心模块，包含了volume操作的业务逻辑
* __Store__：负责存储volume元数据，目前元数据存储在本地的boltdb文件中。
* __Driver__：volume driver接口，抽象了volume相关驱动的基本功能
* __Modules__：具体的volume driver，目前存在local, tmpfs, volume plugin, ceph四种volume驱动



![pouch_volume_arch.png | center | 747x624](https://cdn.yuque.com/lark/0/2018/png/108876/1526824612386-4a990eb9-77b8-4bdf-83ff-a243501a45d3.png "")


VolumeManager是PouchContainer中的存储组件(其他组件包括ContainerManager, ImageManager、NetworkManager等)，它是所有volume操作的入口，目前提供了Create/Remove/List/Get/Attach/Detach接口。Core包含了volume操作的核心逻辑，向下负责调用底层具体的volume driver，实现volume的创建、删除、attach、detach等操作，同时调用Store,实现volume元数据管理。Store模块专门负责volume的元数据管理，volume的相关状态都会通过Store进行存储，之所以将元数据管理专门作为一个模块，是为了将来方便扩展，目前volume元数据是存储在boltdb，未来也可能存入etcd等。Driver抽象了volume driver需要实现的接口，一个具体的volume driver需要实现如下接口：

```go
type Driver interface {
        // Name returns backend driver's name.
        Name(Context) string

        // StoreMode defines backend driver's store model.
        StoreMode(Context) VolumeStoreMode

        // Create a volume.
        Create(Context, *types.Volume, *types.Storage) error

        // Remove a volume.
        Remove(Context, *types.Volume, *types.Storage) error

        // Path returns volume's path.
        Path(Context, *types.Volume) (string, error)
}
```
       

## 2. PouchContainer支持的volume类型

目前，PouchContainer支持三种具体的volume类型，即local, tmpfs和ceph，还通过volume plugin这种通用的存储插件机制支持更多的第三方存储。

### 2.1 local volume

local volume是PouchContainer默认的volume类型，适合存储需要持久化的数据，它的生命周期独立于容器的生命周期。

local volume本质上，是由pouchd在/var/lib/pouch/volume目录下创建的一个子目录。相较于docker，PouchContainer的local volume拥有更多的实用特性，包括：

* 指定挂载目录创建volume
* 可以指定volume大小

首先我们可以指定目录创建一个local volume。该特性在生产中非常实用。对于某些应用，如数据库，我们需要挂载专门的块设备，用于存储数据库数据，例如运维人员将块设备格式化后，挂载到/mnt/mysql\_data目录。执行以下命令，我们就创建了一个挂载在/mnt/mysql\_data的volume，然后可以将该volume挂载到容器指定目录，启动容器。

```powershell
pouch volume create --driver local --option mount=/mnt/mysql_data  --name mysql_data
```

其次，我们可以限制volume的大小。该功能依赖于底层文件系统提供的quato功能，目前支持的底层文件系统为ext4和xfs，同时对内核版本也有要求。

```powershell
pouch volume create --driver local --option size=10G --name test_quota
```

### 2.2 tmpfs volume

tmpfs volume的数据并不会持久化到硬盘中去，只存储于内存中 (若内存不足，则存入swap)，访问速度快，但当容器停止运行时，该volume里面的所有信息都会消失，因此tmpfs volume只适合保存一些临时和敏感的数据。

tmpfs volume默认存储/mnt/tmpfs目录下，你也可以通过 *-o mount* 指定其挂载路径。不过指定tmpfs的挂载路径没有什么意义，因为tmpfs内容直接存储在内存中。

```powershell
pouch volume create --driver tmpfs --name tmpfs_test
```

### 2.3 ceph volume

ceph是一种比较特殊的volume类型，ceph volume是将数据存储到ceph集群(ceph rbd 存储)中，因此可以实现volume跨物理机的迁移。

目前外界暂时不能使用ceph volume。从PouchContainer volume架构图可知，ceph driver和driver层之间还有一层alibaba storage controller（注意：alibaba storage controller只是一个代称），这是阿里巴巴内部的一套容器存储管理平台，后面对接了ceph/pangu/nas等诸多存储方案。PouchContainer通过与该容器存储管理平台对接，可以直接利用ceph提供volume。后期我们可能开源该容器存储管理平台。

### 2.4 volume plugin

volume plugin是一种通用性的volume，准确来说它是一种volume的扩展机制。目前docker通过插件机制可以管理诸多的第三方存储，PouchContainer也实现了该volume plugin机制，可以无缝对接原先docker已经存在的volume plugin。

作为一个volume plugin，必须实现[volume plugin protocal](https://docs.docker.com/engine/extend/plugins_volume/#volume-plugin-protocol)。volume plugin其实本质上是一个web server，该web server实现了如下服务，所有请求均为POST请求。

```plain
/VolumeDriver.Create          // Volume创建服务

/VolumeDriver.Remove          // Volume删除服务

/VolumeDriver.Mount           // Volume挂载服务

/VolumeDriver.Path            // Volume挂载路径服务

/VolumeDriver.Unmount         // Volume卸载服务

/VolumeDriver.Get             // Volume Get服务

/VolumeDriver.List            // Volume List服务

/VolumeDriver.Capabilities    // Volume Driver能力服务
```

## 3. bind mounts与volumes

PouchContainer目前支持两种数据持久化的方式，除了上述的volumes，还可以利用bind mounts。bind mounts，顾名思义，指的是直接将宿主机的目录挂载到容器里面。

```powershell
pouch run -d -t -v /hostpath/data:/containerpath/data:ro ubuntu sh
```

上述这条命令就将宿主机上的/hostpath/data目录已只读的方式挂载到容器的/containerpath/data目录下。

bind mounts依赖于宿主机文件系统目录结构，而volume，在PouchContainer中有专门的机制进行管理。volumes相对于bind mounts，有以下优势：

* volumes相对于bind mounts，更容易进行备份和管理；
* PouchContainer提供了专门的cli和api，用来管理volumes；
* volumes适合在多个容器之间安全地共享；
* volumes提供了插件机制，可以更加方便地对接第三方存储。

## 4. PouchContainer volume未来的发展

[CSI](https://github.com/container-storage-interface/spec)，即Container Storage Interface（该项目定义了容器调度层和容器之间的存储接口），目前已发布了v0.2版本。Pouch未来可能增加一种通用类型的driver，用于对接已实现CSI接口的存储系统。

## 5. 总结

本文介绍了PouchContainer的volume机制， volume机制主要是为了解决pouch容器数据持久化的问题PouchContainer，目前支持local,tmpfs,ceph三种driver，同时支持以volume plugin的形式对接更多的第三方存储。

