# PouchContainer 与 Disk quota

## 什么是 Disk quota

Disk quota 是一种限制文件系统磁盘用量的技术。PouchContainer 使用 disk quota 来限制文件系统磁盘空间。众所周知，基于块设备的方法能够通过设定块设备的大小，轻松直接地帮助限制磁盘空间的使用，然而基于文件系统的方式很难达成这一点。Disk quota旨在限制文件系统磁盘使用。 目前 PouchContainer 支持以graph driver为基础的OverlayFS。

目前的底层文件系统中仅有 ext4 和 xfs 支持 disk quota。此外，有三种实现方法：**用户配额**，**组配额**和**项目配额**。

限制磁盘用量的两个维度：

* usage quota（block quota）：为文件系统设置磁盘用量（并非 inode number）；
* file quota (inode quota)：限制文件或 inode 的分配。

PouchContainer 目前仅支持 block quota，暂时还不支持 inode。

## PouchContainer 中的 Disk quota 

PouchContainer 中的 disk quota 依赖于 PouchContainer 运行的内核版本。下表描述了每个文件系统对 disk quota 的支持：

|| user/group quota | project quota|
|:---:| :----:| :---:|
|ext4| >= 2.6|>= 4.5|
|xfs|>= 2.6|>= 3.10|

虽然相关内核版本中的每个文件系统都支持disk quota，用户还是需要安装 [quota-tools-4.04](https://nchc.dl.sourceforge.net/project/linuxquota/quota-tools/4.04/quota-4.04.tar.gz)。
此配额工具尚未打包到 PouchContainer 的 rpm 中。我们会在将来完成该工具。

## 入门

PouchContainer中有两种方法可以让容器参与底层文件系统。一个是 container rootfs，另一个是从主机（容器外部）绑定的 container volume 。PouchContainer 同时涵盖了这两个维度。

### Container Rootfs Disk quota

用户可以为已创建的 container rootfs 设置标志 `--disk-quota` 以限制磁盘空间使用情况，例：`--disk-quota 10g`。设置成功后，我们可以通过命令 `df -h` 看到rootfs的大小是10GB。这表明disk quota 已经生效了。

```bash
$ pouch run -ti --disk-quota 10g registry.hub.docker.com/library/busybox:latest df -h
Filesystem                Size      Used Available Use% Mounted on
overlay                  10.0G     24.0K     10.0G   0% /
tmpfs                    64.0M         0     64.0M   0% /dev
shm                      64.0M         0     64.0M   0% /dev/shm
tmpfs                    64.0M         0     64.0M   0% /run
tmpfs                    64.0M         0     64.0M   0% /proc/kcore
tmpfs                    64.0M         0     64.0M   0% /proc/timer_list
tmpfs                    64.0M         0     64.0M   0% /proc/sched_debug
tmpfs                     1.9G         0      1.9G   0% /sys/firmware
tmpfs                     1.9G         0      1.9G   0% /proc/scsi
```

### Volume Disk quota

用户还可以在创建 volume 时设置其磁盘配额。可以通过添加 `--option` 或 `-o` 标志轻松指定所需的磁盘空间限制数字，例：`-o size = 10g`。


创建 disk quota limited volume 后，用户可以将其绑定到运行容器。以下示例执行了命令：
`pouch run -ti -v volume-quota-test：/ mnt registry.hub.docker.com/library/busybox:latest df -h`。在运行容器中，目录 `/ mnt` 的大小被限制为10GB。

```bash
$ pouch volume create -n volume-quota-test -d local -o mount=/data/volume -o size=10g
Name:         volume-quota-test
Scope:
Status:       map[mount:/data/volume sifter:Default size:10g]
CreatedAt:    2018-3-24 13:35:08
Driver:       local
Labels:       map[]
Mountpoint:   /data/volume/volume-quota-test

$ pouch run -ti -v volume-quota-test:/mnt registry.hub.docker.com/library/busybox:latest df -h
Filesystem                Size      Used Available Use% Mounted on
overlay                  20.9G    212.9M     19.6G   1% /
tmpfs                    64.0M         0     64.0M   0% /dev
shm                      64.0M         0     64.0M   0% /dev/shm
tmpfs                    64.0M         0     64.0M   0% /run
/dev/sdb2                10.0G      4.0K     10.0G   0% /mnt
tmpfs                    64.0M         0     64.0M   0% /proc/kcore
tmpfs                    64.0M         0     64.0M   0% /proc/timer_list
tmpfs                    64.0M         0     64.0M   0% /proc/sched_debug
tmpfs                     1.9G         0      1.9G   0% /sys/firmware
tmpfs                     1.9G         0      1.9G   0% /proc/scsi
```
<!--对于科技文章，它包含大量的英文专有名词，很多时候这些全新的科技术语还没有对应的中文翻译，或者其中文翻译并不是开发者最常用的表达方式。比如文章中的“quota”，并没有太多的开发者在其文档或论坛中使用“配额”一词来代替“quota”。 所以在翻译文档的时候我们还是要注意各国开发者习惯的词汇用法。-->