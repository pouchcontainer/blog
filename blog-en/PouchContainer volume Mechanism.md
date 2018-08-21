PouchContainer volume is a mechanism specifically designed to solve data persistence of container. To understand the mechanism of the volume, you need to understand the imaging mechanism of PouchContainer. PouchContainer, just like Docker, implements the layering mechanism of the image. Image layering mechanism refers to that the container image is actually superimposed by multiple read-only image layers so that different image can reuse the image layer, which significantly increases the container dispatching efficiency and reduces the container start time. When the container is started, pouchd(PouchContainer daemon) will add a read-write layer at the top level of the startup image and all following read-write actions of the container would be recorded in this layer. It also brings a problem, which is data persistence of container. If the container is deleted, all previous edit to the container is lost when started through that mirror, which is fatal to the application with states,like the database.

Volume bypasses image mechanism, allowing the container data to exist on the host as a normal file or directory. When the container is stopped or deleted, the data in volume would not be affected, thus the data persistence is implemented and the volume data can be shared among different containers.

## 1. PouchContainer volume Overall Structure

This part of the content may involve the source code implementation of volume in PouchContainer.

The overall structure of PouchContainer volume is mainly formed by the following components:


* __VolumeManager__：this component is the main entrance for volume actions
* __Core__：Core is core module of volume, which contains the business logic of volume action.
* __Store__：stores the metadata of volume. The metadata is stored in the local boltdb directory.
* __Driver__：volume driver interface，which defines the abstract functions of volume drivers
* __Modules__： specific volume driver，there are four types of volume drivers for now, local, tmpfs, volume plugin and ceph.


![pouch_volume_arch.png | center | 747x624](https://cdn.yuque.com/lark/0/2018/png/108876/1526824612386-4a990eb9-77b8-4bdf-83ff-a243501a45d3.png "")

VolumeManager is a storage component of PouchContainer(Other components include ContainerManager, ImageManager、NetworkManager, etc) and the entrance for all volume action. It provides Create/Remove/List/Get/Attach/Detach interface. The Core includes the core logic of the volume action, calls the underlying specific volume driver and implements actions, including create, remove, attach, detach, etc. It also implements volume metadata management with calling Store. Store module is specifically designed got volume metadata management. All state of the volume is stored through Store. 
Volume metadata is stored in boltdb currently and may be stored into etcd in the future. Driver defines the abstract interface that a volume driver needs to implement. A volume driver needs to implements the following interface:

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

## 2. the volume type that PouchContainer support 
So far, PouchContainer supports three specific types: local, tmpfs and ceph. It supports more third-party storages through universal storage plug-in mechanism, volume plug-in

### 2.1 local volume

Local volume is a default type in PouchContainer which is suited to store data with persistence. Its lifecycle is independent of the container’s lifecycle. 

When a volume is created, the default driver type would be local if the driver type is not specified. Local volume is essentially created by pouchd in the subdirectory under /var/lib/pouch/volume directory. Comparing with Docker, Pouch Container’s local volume have more usability features such as:

* Specify the mounting directory to create volume 
* Specify the size of the volume

First, we can specify a directory and create a local volume. It is a useful feature in production. For some applications, such as the database, we need to mount special block devices to store data in the database. For example, after formatting block devices, operation staffs will mount it under /mnt/mysql_data directory. By executing command line below, we will create a volume mounted under /mnt/mysql_data directory. Then we can mount this volume under a specified directory and start the volume.

```powershell
pouch volume create --driver local --option mount=/mnt/mysql_data  --name mysql_data
```

Besides, we can limit the size of the volume. This function depends on quato function that provided by the underlying file system. So far, ext4 and xfs are two underlying file systems that support this function. There is also the requirement for kernel version.

```powershell
pouch volume create --driver local --option size=10G --name test_quota
```

### 2.2 tmpfs volume

Data in tmpfs volume is not persistent in hardware, and it is only stored in the internal storage (if there are not enough storage, then store the data into swap). It has high access speed. But when the volume stops running, all information in the volume will disappear. As a result, tmpfs volume is only suitable for storing some temporary and sensitive data. 

Tmfs volume is stored under /mnt/tmpfs by default, you can also specify its mounting directory via –o mount. However, specifying the mounting directory is meaningless, since the content of tmpfs is directly stored in the storage.

```powershell
pouch volume create --driver tmpfs --name tmpfs_test
```

### 2.3 ceph volume

ceph is a special volume type. Ceph volume stores data in ceph cluster(ceph rbd storage), so it can achieve migration across physical machines. 

So far, ceph volume is not usable from externals. From PouchContainer architecture diagram, there is a layer for Alibaba storage controller (notice: Alibaba storage controller is only an antonomasia) between ceph driver and driver layer. This is a set of container storage management platforms inside Alibaba, followed by a number of storage solutions such as ceph/pangu/nas. PouchContainer can directly use ceph to provide volume by docking with the container storage management platform. Later we may open up the container storage management platform.

### 2.4 volume plugin

volume plugin is a versatile volume, which is the expansion mechanism of volume to be precise. At present, Docker can manage many third-party storages through the plug-in mechanism. PouchContainer also implements the volume plugin mechanism, which can seamlessly connect to the existing volume plugin of the existing Docker. 

As a volume plugin, you must realize [volume plugin protocal](https://docs.docker.com/engine/extend/plugins_volume/#volume-plugin-protocol). The volume plugin is essentially a web server, this web server implements following services (all requests are POST requests):

```plain
/VolumeDriver.Create          // Volume Creation Service

/VolumeDriver.Remove          // Volume Deletion Service

/VolumeDriver.Mount           // Volume Mounting Service

/VolumeDriver.Path            // Volume Mounting Path Service

/VolumeDriver.Unmount         // Volume Unmounting Service

/VolumeDriver.Get             // Volume Get Service

/VolumeDriver.List            // Volume List Service

/VolumeDriver.Capabilities    // Volume Driver Capability Service
```

## 3. bind mounts and volumes

PouchContainer currently supports two ways of data persistence. In addition to the above volumes, you can also use bind mounts. Bind mounts, as the name suggests, refers to directly mounting the host's directory to the container. 

```powershell
pouch run -d -t -v /hostpath/data:/containerpath/data:ro ubuntu sh
```

The above command line will mount the /hostpath/data directory on the host to the /containerpath/data directory of the container in a read-only manner. 

Bind mounts depend on the host file system directory structure, while volume has a special mechanism for management in PouchContainer. Volumes have the following advantages over bind mounts:

* volumes, comparing to bind mounts, are easier to backup and manage;
* PouchContainer provides exclusive cli and api to manage volumes;
* volumes are suited for sharing among multiple containers.
* volumes provide the plug-in mechanism to use third-party storage.


## 4. Future of PouchContainer volume

[CSI](https://github.com/container-storage-interface/spec)，which is Container Storage Interface(This project defines the storage interface between container scheduling layer and container), has published v0.2 version. The Pouch may add a general type driver, which is used for the storage system with CSI interface implemented.

## 5.Conclusion

This document introduces the volume mechanism of PouchContainer， which is designed for data persistence of PouchContainer. It supports local,tmpfs,ceph drivers for now, and supports third-party storage in the form of volume plugin.