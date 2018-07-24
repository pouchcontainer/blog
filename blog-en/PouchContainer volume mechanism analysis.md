> PouchContainer volume is a mechanism specifically designed to solve the data persistence of the container. To understand the mechanism of the volume, you may need to understand the image mechanism of PouchContainer.

> PouchContainer, like Docker, implements a layered mechanism for image. The so-called mirror layering mechanism means that the mirror image of the container is actually superimposed by multiple read-only mirror layers, so that different mirror images can reuse the mirror layer, which greatly speeds up the efficiency of image distribution and reduces container startup time.

> When the container needs to be started, pouchd (pouchd mentioned below refers to the PouchContainer daemon) will add a read-write layer at the top level of the boot image, and all subsequent read and write operations in the container will be recorded in the read-write layer.
However, this also introduces a problem, that is, HOW-TO persist container data. If we delete the container and start it again through the image, the history changes about the container are lost, which is fatal for stateful applications such as databases.

> Volume bypasses the mirroring mechanism, so that the data in the container are  stored on the host machine in the form of a normal file or directory. 

> The data in the volume will not be affected even though the container is stopped or deleted, thereby implementing container data persistence feature. Moreover, volume data can be shared between different containers.

## 1. PouchContainer Volume Overview

This part of the content may involve PouchContainer Volume source code.

The PouchContainer volume overall architecture currently consists of the following components:

- **VolumeManager** : the entrance of volume related operations.
- **Core**: the core module of volume, which contains the business logic of the volume operation.
- **Store**: the module be responsible for storing volume meta data, and the meta data is currently stored in the local boltdb file.
- **Driver**: the interfaces of volume driver,  which contains volume related driver basic feature convention.
- **Module**: the specific volume driver, there currently exists four volume drives: local, tmpfs, volume plugin, and ceph.

![pouch_volume_arch.png | center | 747x624](https://cdn.yuque.com/lark/0/2018/png/108876/1526824612386-4a990eb9-77b8-4bdf-83ff-a243501a45d3.png "")

`VolumeManager` is a storage component in `PouchContainer` (other components include`ContainerManager`, `ImageManager`, `NetworkManager`, etc.), which is the entry point for all volume related operations. Currently, the `Create`, `Remove`, `List`, `Get`, `Attach `and `Detach` interface is provided.

The `Core` module contains the core implementation of the volume operation. It is responsible for calling the underlying specific volume driver to  execute the `Creat`, `Delete`, ` Attach`and `Detach` operations and  manages volume metadata by calling the `Store` module.

The `Store`module is specifically responsible for the volume metadata management. The relevant state of the volume will be stored through the `Store` module. The reason why the metadata management is designed as a single module is to extend easily in the future. Currently, the volume metadata is stored in the boltdb file, and may introduce `etcd` in the future.

`Driver` module abstracts the interface that the volume driver needs to implement. A specific volume driver needs to implement the following interfaces:

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

## 2. supported volume type

PouchContainer currently supports three specific volume types, namely local volume, tmpfs volume and ceph volume. PouchContainer also supports more third-party storage through the volume plugin.


### 2.1 local volume

The default volume type of PouchContainer is local volume, which is proper for storing persistent data and has an independent life cycle from that of the Pouchcontainer.

The local volume is essentially a subdirectory created by ` pouchd` under `/var/lib/pouch/volume` directory. PouchContainer's local volume has more userful features than docker's does, including:

* creating volume under specific mount directory
* specifying the size fo volume

Firstly, a local volume can be created under a specified local volume. This feature is quite practical in production. In applications such as database, a specialized block device is needed for storing database data. Usually, a block device can be formatted and then mounted to a specific directory. For example, by executing the following instruction, a new volume mounted to directory `/mnt/mysql\_data` will be created, then the created volume will be mounted to specific container directory, finally container initialized.


```powershell
pouch volume create --driver local --option mount=/mnt/mysql_data  --name mysql_data
```

Secondly, the size of volume can be limited. This feature relies on the quato functionality of the underlying file system. For now, the supported file systems include ext4 and xfs, specific kernel versions of which are also required.

```powershell
pouch volume create --driver local --option size=10G --name test_quota
```

### 2.2 tmpfs volume

Data in tmpfs volume stays only in memory(or swap when memory is insufficient). This provides high access speed,  however all the data is going to disapper when container terminates. Therefore,  `tmpfs` volume is only proper for storing some temporary or sensitive data.

The `tmpfs` volume is placed by default under the `/mnt/tmpfs` directory. The mount path can also be specified with option *-o mount*. However, specifying mount path for `tmpfs` volume makes no sense because `tmpfs` volume stores data directly in memory.

```powershell
pouch volume create --driver tmpfs --name tmpfs_test
```
### 2.3 ceph volume

`ceph` volume is a special type of volume where data is stored in a `ceph` cluster(`ceph` rbd storage), making the migration of volumes across physical machines possible.

`ceph` volume is unavailale at present. From the PouchContainer volume architecture diagram, there is a layer of alibaba storage controller between the ceph driver and the parent driver layer, which is an alibaba internal container storage management platform, docking many storage solutions such as ceph, pangu and nas. PouchContainer can directly use `ceph` to provide volume by docking with the container storage management platform. The container storage management platform is going to be open sourcing in the future. 


### 2.4 volume plugin


Volume plugin is a general-purpose volume, which is exactly a volume extension mechanism. At present, docker can manage many third-party storage through the plug-in mechanism. PouchContainer also implements the volume plugin mechanism, which can seamlessly connect to the existing volume plugin of the existing docker.

A volume plugin should implement [volume plugin protocal](https://docs.docker.com/engine/extend/plugins_volume/#volume-plugin-protocol)。The volume plugin is essentially a web server who implements the following services(all requests are POST requests):

```plain
/VolumeDriver.Create          

/VolumeDriver.Remove          

/VolumeDriver.Mount          

/VolumeDriver.Path            

/VolumeDriver.Unmount        

/VolumeDriver.Get            

/VolumeDriver.List            
/VolumeDriver.Capabilities    
```
## 3. bind mounts and volumes

PouchContainer currently supports two methods for data persistence. In addition to the above` volume` approach,  you can also adapt **bind mounts** approach,  which implies directly mounting the host's directory to the container.

```powershell
pouch run -d -t -v /hostpath/data:/containerpath/data:ro ubuntu sh
```
The above command will mount the `/hostpath/data` directory on the host to the `/containerpath/data` directory of the container in a read-only manner.

Binding mount approach depends on the underlaying host file system directory structure,  while volume approach provided by `PouchContainer`has a special mechanism to manage these differences.

volume approach has the following advantages over binding mounts:

- volumes are easier for back up and management than binding mounts.
- `PouchContainer` provides specialized `CLI` and `API` for managing volumes.
- volumes are suitable for secure sharing between multiple containers.
- volumes provide a plugin mechanism for docking easily  third party storage.

## 4. Future work on PouchContainer volume

[CSI](https://github.com/container-storage-interface/spec), the Container Storage Interface (which defines the storage interface between the container scheduling layer and the container), has been released for version 0.2. 

In the future, Pouch may add a generic type of driver for docking storage systems that have implemented `CSI` interfaces.

## 5. Summary

This article introduces the volume mechanism in `PouchContainer`. The volume mechanism is mainly to solve data persistence problems of pouch container. `PouchContainer` currently supports `local`, `tmpfs` and `ceph` driver,  moreover, `PouchContainer `  also supports more  third-party storage by volume plugins.