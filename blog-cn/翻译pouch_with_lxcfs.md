# PouchContainer与LXCFS

容器技术为不同的隔离环境提供传统的虚拟化技术,如VMware，KVM。普遍的Linux容器以削弱隔离为代价换来了更快的容器打包以及设定速度。容器的资源视图是Linux容器面临的最广为人知的隔离问题之一。

容器为用户提供了一种限制运行容器的资源使用的大致方法，资源包括内存，cpu, blkio等。但容器中的步骤无法使之获取额外的相应资源。但是，如果容器内的步骤用命令 `free`, `cat /proc/meminfo`, `cat /proc/cpuinfo`, `cat /proc/uptime`查出资源上限，这些命令会得到对宿主机而言正确的信息，而不是容器本身。

举个例子来说，如果我们创建一个200MB主机内存，总内存2GB的容器，我们可以发现通过命令`free`得出的资源上限是不正确的，结果是主机的总内存：

``` shell
$ pouch run -m 200m registry.hub.docker.com/library/ubuntu:16.04 free -h
              total        used        free      shared  buff/cache   available
Mem:           2.0G        103M        1.2G        3.3M        684M        1.7G
Swap:          2.0G          0B        2.0G
```

## 资源隔离情景

由于缺乏资源隔离，一些应用可能会在容器提供的环境中运行异常。应用可能发觉它的运行时间与在正常物理机或虚拟机上的运行时间不同。以下是一些由于隔离匮乏而影响应用运行的例子:

> 对于很多基于JVM的Java应用，应用的启动脚本多依赖于系统资源对于JVM分配堆栈大小的能力。因此，一个在2GB内存主机上创建出的200MB内存的容器，会认为它能够使用2GB的内存，所以启动脚本会让Java运行时根据与200MB上限相差甚远的2GB内存分配堆栈大小。因此应用程序当然会启动失败。对于Java应用，一些库会根据它们的资源视图分配堆栈大小，这也会暴露出潜在的安全问题。

除了内存资源视图的问题，还有弱CPU资源视图隔离的问题。

> 大多数中间软件会根据它认为的处理器信息去设置默认的线程数量。用户会配置在cgroup文件生效的容器cpu是不无道理的事情。但是，容器中的步骤通过使用`/proc/cpuinfo` 总是得到整个内核数量，并一定会导致不稳定。

资源隔离会影响容器里系统级的应用程序。

> 容器还可以用来打包系统级的应用程序，并且系统级应用程序会时常需要通过虚拟文件系统`/proc`获取系统信息。如果使用的是主机的正常运行时间，而不是容器的正常运行时间，系统级应用程序会失去控制或者不按预期运行。`cpuinfo` 和 `meminfo`，以及一些其它系统资源视图也是这些应用程序需要考虑的方面。

## 什么是LXCFS

[LXCFS](https://github.com/lxc/lxcfs)是一个小型的 [FUSE文件系统](https://en.wikipedia.org/wiki/Filesystem_in_Userspace) ，开发它的意图是让Linux容器更像一个虚拟机。最开始它是LXC的副产品，但其实它可以在任何运行环境被使用。 LXCFS 兼容Linux kernel 2.6+。LXCFS可以处理 `procfs`里重要文档提供的信息，包括:

* /proc/cpuinfo
* /proc/diskstats
* /proc/meminfo
* /proc/stat
* /proc/swaps
* /proc/uptime

PouchContainer在之前的版本中一直支持LXCFS，并且一直很稳定。 换句话来说，如果用户启动LXCFS,LXCFS在主机上会运行守护进程。通常来说，当创建了一个有资源上限的容器，一些映射这个容器的虚拟文件会在cgroup文件系统中被创建。然后，LXCFS会动态地读取这些文件里的数值，比如`memory.limit_in_bytes`，并在主机上产生一个新的分支虚拟文件（如 `/var/lib/lxc/lxcfs/proc/meminfo`），然后把这个文件绑定在容器上。最后，容器里的步骤会通过类似`/proc/meminfo`的命令读取文件，获取到真正的资源视图。 

下面是LXCFS和容器的结构：

![pouch_with_lxcfs](../static_files/pouch_with_lxcfs.png)

## 开始使用

利用LXCFS，用户达到资源隔离是很容易的事情。事实上，如果LXCFS软件在$PATH中不存在的话，它会自动与PouchContainer一起安装在主机上。

在体验LXCFS确保的资源视图隔离之前, 用户需要确认LXCFS模式在pouchd中是被允许的。如果LXCFS模式还没有被设置好，用户需要中止pouchd，再通过 `pouchd --enable-lxcfs`命令开启pouchd。只有在pouchd开启LXCFS模式，用户才能在容器中使用LXCFS功能。

当pouchd中开启着LXCFS模式时，pouchd有额外能力可以创建有隔离的资源视图的容器。除此之外，pouchd还可以创建没有资源视图隔离的普通容器。

最后，命令`pouch run`中的 `--enableLxcfs` 标志是在已经允许LXCFS模式的pouchd中使得LXCFS可以创建容器的唯一方法。在这里我们在2GB内存的主机上创建一个内存为200MB的容器。

### 启动条件

如果是在Centos中，首先要确保你的lxcfs服务在运行。 如果是其他的系统可能会使用不同的启动方式。


```
$ systemctl start lxcfs
$ ps -aux|grep lxcfs
root     1465765  0.0  0.0  95368  1844 ?        Ssl  11:55   0:00 /usr/bin/lxcfs /var/lib/lxcfs/
root     1465971  0.0  0.0 112736  2408 pts/0    S+   11:55   0:00 grep --color=auto lxcfs
```

通过更改--enable-lxcfs标记启动 pouchd lxcfs

```
$ cat /usr/lib/systemd/system/pouch.service
[Unit]
Description=pouch

[Service]
ExecStart=/usr/local/bin/pouchd --enable-lxcfs
...

$ systemctl daemon-reload && systemctl restart pouch
```

``` shell
$ pouch run -m 200m --enableLxcfs registry.hub.docker.com/library/ubuntu:16.04 free -h
              total        used        free      shared  buff/cache   available
Mem:           200M        876K        199M        3.3M         12K        199M
Swap:          2.0G          0B        2.0G
```

我们可以发现返回的总内存大小就是容器的内存上限。
在执行上述命令过后，我们同时也可以发现在容器中的进程获取的资源也是它实际的资源上限。换句话说，在容器中的应用会更加的安全。这也是PouchContainer重要的特性之一。

link: https://github.com/alibaba/pouch/blob/master/docs/features/pouch_with_lxcfs.md
