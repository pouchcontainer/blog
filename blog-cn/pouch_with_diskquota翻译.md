# 具备Diskquota的PouchContainer

## 什么是diskquota

Diskquota是一种用于限制文件系统磁盘使用的技术。PouchContainer使用diskquota来限制文件系统磁盘空间。众所周知，基于块设备的方式，通过设置块设备的大小，可以直接使限制磁盘空间使用变得容易。但是使用对于基于文件系统的方式实现限制磁盘空间使用却很难。Diskquota的诞生正是用于限制文件系统磁盘文件。目前，PouchContainer支持基于图像驱动覆盖文件系统的diskquota。

目前，底层的文件系统中只有ext4和xfs支持diskquota。此外，现在有三种实现方式:**user quota**, **group quota** and **project quota**.

对于磁盘使用的限制，有两个规模:

- 使用配额(块配额): 设置文件系统目录的磁盘使用(不用于索引节点号);
- 文件配额(索引节点配额): 限制文件或索引节点分配。

PouchContainer目前只支持块配额，暂时不支持索引节点配额。

## PouchContainer中的Diskquota

PouchContainer中的Diskquota依赖于PouchContainer运行的内核版本。下表描述来每个文件系统支持的Diskquota。

| | user/group quota | project quota 
:-: | :-: | :-: 
ext4 | \>= 2.6| \>= 4.5 
xfs | \>= 2.6 | \>= 3.10

尽管每个文件系统在相关内核版本下支持diskquota，但是用户仍然需要安装[quota-tools-4.04](https://nchc.dl.sourceforge.net/project/linuxquota/quota-tools/4.04/quota-4.04.tar.gz)。
这个配额工具还没有打包在PouchContainer软件包管理器中。我们会在未来将其完成。

## 开始

PouchContainer为容器涉及底层文件系统提供了两种方式。一个是容器根文件系统，另一个是从主机（容器外）到内部的容器卷标绑定。两个维度都包含在PouchContainer。

### 容器根文件系统diskquota

用户可以为一个以创建的容器根文件系统，设置标志`--disk-quota`来限制磁盘空间的使用，例如`--disk-quota 10g`。成功设置这个标志后，我们可以通过命令`df -h`得到根文件系统的大小是10G。 同时，这说明diskquota生效了。

```
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

### 卷标diskquota

用户也可以在创建时设置卷标的磁盘限制。通过增加一个`--option` 或 `-o` 标志，指定磁盘空间限制为一个期望的数据变得十分容易，例如 `-o size=10g`。

创建限制卷标的diskquota后，用户可以将这个卷标绑定到一个正在执行的容器。在下面的例子中，执行命令
`pouch run -ti -v volume-quota-test:/mnt registry.hub.docker.com/library/busybox:latest df -h`。
在运行着的容器中，目录`/mnt` 被限制到10GB的大小。

```
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
