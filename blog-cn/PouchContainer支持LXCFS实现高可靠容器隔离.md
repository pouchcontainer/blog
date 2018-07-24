# PouchContainer 支持 LXCFS 实现高可靠容器隔离

## 引言
PouchContainer 是 Alibaba 开源的一款容器运行时产品，当前最新版本是 0.3.0，代码地址位于：[https://github.com/alibaba/pouch](https://github.com/alibaba/pouch)。PouchContainer 从设计之初即支持 LXCFS，实现高可靠容器隔离。Linux 使用 cgroup 技术实现资源隔离，然而容器内仍然挂载宿主机的 /proc 文件系统，用户在容器内读取 /proc/meminfo 等文件时，获取的是宿主机的信息。容器内缺少的 `/proc 视图隔离`会带来一系列的问题，进而拖慢或阻碍企业业务容器化。LXCFS ([https://github.com/lxc/lxcfs](https://github.com/lxc/lxcfs)) 是开源 FUSE 文件系统，用以解决 `/proc 视图隔离`问题，使容器在表现层上更像传统的虚拟机。本文首先介绍 LXCFS 适用业务场景，然后简要介绍 LXCFS 在 PouchContainer 内部集成的工作。

## LXCFS 业务场景
在物理机和虚拟机时代，公司内部逐渐形成了自己的一套工具链，诸如编译打包、应用部署、统一监控等，这些工具已经为部署在物理机和虚拟机中的应用提供了稳定的服务。接下来将从监控、运维工具、应用部署等方面详细阐述 LXCFS 在上述业务容器化过程中发挥的作用。

### 监控和运维工具
大部分的监控工具，依赖 /proc 文件系统获取系统信息。以阿里巴巴为例，阿里巴巴的部分基础监控工具是通过 tsar（[https://github.com/alibaba/tsar](https://github.com/alibaba/tsar)) 收集信息。而 tsar 对内存、CPU 信息的收集，依赖 /proc 文件系统。我们可以下载 tsar 的源码，查看 tsar 对 /proc 目录下一些文件的使用：

```
$ git remote -v
origin	https://github.com/alibaba/tsar.git (fetch)
origin	https://github.com/alibaba/tsar.git (push)
$ grep -r cpuinfo .
./modules/mod_cpu.c:    if ((ncpufp = fopen("/proc/cpuinfo", "r")) == NULL) {
:tsar letty$ grep -r meminfo .
./include/define.h:#define MEMINFO "/proc/meminfo"
./include/public.h:#define MEMINFO "/proc/meminfo"
./info.md:内存的计数器在/proc/meminfo,里面有一些关键项
./modules/mod_proc.c:    /* read total mem from /proc/meminfo */
./modules/mod_proc.c:    fp = fopen("/proc/meminfo", "r");
./modules/mod_swap.c: * Read swapping statistics from /proc/vmstat & /proc/meminfo.
./modules/mod_swap.c:    /* read /proc/meminfo */
$ grep -r diskstats .
./include/public.h:#define DISKSTATS "/proc/diskstats"
./info.md:IO的计数器文件是:/proc/diskstats,比如:
./modules/mod_io.c:#define IO_FILE "/proc/diskstats"
./modules/mod_io.c:FILE *iofp;                     /* /proc/diskstats*/
./modules/mod_io.c:    handle_error("Can't open /proc/diskstats", !iofp);
```

可以看到，tsar 对进程、IO、CPU 的监控都依赖 /proc 文件系统。

当容器内 /proc 文件系统提供的是宿主机资源信息时，这类监控不能监控容器内信息。为了满足业务需求，需要适配容器监控，甚至需要单独为容器内监控开发另一套监控工具。这种改变势必会拖慢甚至阻碍企业现存业务容器化的步伐，容器技术要尽可能兼容公司原有的工具链，兼顾工程师的使用习惯。

PouchContainer 支持 LXCFS 可以解决上述问题，依赖 /proc 文件系统的监控、运维工具，部署在容器内或宿主机上对工具是透明的，现存监控、运维工具无需适配或重新开发，即可平滑迁移到容器内，实现容器内的监控和运维。

接下来让我们从实例中直观感受一下，在一台 Ubuntu 虚拟机中安装 PouchContainer 0.3.0 :

```
# uname -a
Linux p4 4.13.0-36-generic #40~16.04.1-Ubuntu SMP Fri Feb 16 23:25:58 UTC 2018 x86_64 x86_64 x86_64 GNU/Linux
```

systemd 拉起 pouchd ，默认不开启 LXCFS，此时创建的容器无法使用 LXCFS 的功能，我们看一下容器内相关 /proc 文件的内容：

```
# systemctl start pouch
# head -n 5 /proc/meminfo
MemTotal:        2039520 kB
MemFree:          203028 kB
MemAvailable:     777268 kB
Buffers:          239960 kB
Cached:           430972 kB
root@p4:~# cat /proc/uptime
2594341.81 2208722.33
# pouch run -m 50m -it registry.hub.docker.com/library/busybox:1.28
/ # head -n 5 /proc/meminfo
MemTotal:        2039520 kB
MemFree:          189096 kB
MemAvailable:     764116 kB
Buffers:          240240 kB
Cached:           433928 kB
/ # cat /proc/uptime
2594376.56 2208749.32
```

可以看到，在容器内看到的 /proc/meminfo、uptime 文件的输出与宿主机一致，虽然启动容器的时候指定了内存为 50M，/proc/meminfo 文件并未体现出容器内的内存限制。

在宿主机内启动 LXCFS 服务，手动拉起 pouchd 进程，并指定相应的 LXCFS 相关参数：

```
# systemctl start lxcfs
# pouchd -D --enable-lxcfs --lxcfs /usr/bin/lxcfs >/tmp/1 2>&1 &
[1] 32707
# ps -ef |grep lxcfs
root       698     1  0 11:08 ?        00:00:00 /usr/bin/lxcfs /var/lib/lxcfs/
root       724 32144  0 11:08 pts/22   00:00:00 grep --color=auto lxcfs
root     32707 32144  0 11:05 pts/22   00:00:00 pouchd -D --enable-lxcfs --lxcfs /usr/bin/lxcfs
```

启动容器，获取相应的文件内容：

```
# pouch run --enableLxcfs -it -m 50m registry.hub.docker.com/library/busybox:1.28
/ # head -n 5 /proc/meminfo
MemTotal:          51200 kB
MemFree:           50804 kB
MemAvailable:      50804 kB
Buffers:               0 kB
Cached:                4 kB
/ # cat /proc/uptime
10.00 10.00
```

使用 LXCFS 启动的容器，读取容器内 /proc 文件，可以得到容器内的相关信息。

### 业务应用
对于大部分对系统依赖较强的应用，应用的启动程序需要获取系统的内存、CPU 等相关信息，从而进行相应的配置。当容器内的 /proc 文件无法准确反映容器内资源的情况，会对上述应用造成不可忽视的影响。

例如对于一些 Java 应用，也存在启动脚本中查看 /proc/meminfo 动态分配运行程序的堆栈大小，当容器内存限制小于宿主机内存时，会发生分配内存失败引起的程序启动失败。对于 DPDK 相关应用，应用工具需要根据 /proc/cpuinfo 获取 CPU 信息，得到应用初始化 EAL 层所使用的 CPU 逻辑核。如果容器内无法准确获取上述信息，对于 DPDK 应用而言，则需要修改相应的工具。

## PouchContainer 集成 LXCFS
PouchContainer 从 0.1.0 版开始即支持 LXCFS，具体实现可以参见: [https://github.com/alibaba/pouch/pull/502](https://github.com/alibaba/pouch/pull/502) .

简而言之，容器启动时，通过-v 将宿主机上 LXCFS 的挂载点 /var/lib/lxc/lxcfs/proc/ 挂载到容器内部的虚拟 /proc 文件系统目录下。此时在容器内部 /proc 目录下可以看到，一些列proc文件，包括 meminfo, uptime, swaps, stat, diskstats, cpuinfo 等。具体使用参数如下：

```
-v /var/lib/lxc/:/var/lib/lxc/:shared
-v /var/lib/lxc/lxcfs/proc/uptime:/proc/uptime 
-v /var/lib/lxc/lxcfs/proc/swaps:/proc/swaps 
-v /var/lib/lxc/lxcfs/proc/stat:/proc/stat 
-v /var/lib/lxc/lxcfs/proc/diskstats:/proc/diskstats 
-v /var/lib/lxc/lxcfs/proc/meminfo:/proc/meminfo 
-v /var/lib/lxc/lxcfs/proc/cpuinfo:/proc/cpuinfo
```

为了简化使用，pouch create 和 run 命令行提供参数 `--enableLxcfs`, 创建容器时指定上述参数，即可省略复杂的 `-v` 参数。

经过一段时间的使用和测试，我们发现由于lxcfs重启之后，会重建proc和cgroup，导致在容器里访问 /proc 出现 `connect failed` 错误。为了增强 LXCFS 稳定性，在 PR:[https://github.com/alibaba/pouch/pull/885](https://github.com/alibaba/pouch/pull/885) 中，refine LXCFS 的管理方式，改由 systemd 保障，具体实现方式为在 lxcfs.service 加上 ExecStartPost 做 remount 操作，并且遍历使用了 LXCFS 的容器，在容器内重新 mount。

## 总结
PouchContainer 支持 LXCFS 实现容器内 /proc 文件系统的视图隔离，将大大减少企业存量应用容器化的过程中原有工具链和运维习惯的改变，加快容器化进度。有力支撑企业从传统虚拟化到容器虚拟化的平稳转型。
