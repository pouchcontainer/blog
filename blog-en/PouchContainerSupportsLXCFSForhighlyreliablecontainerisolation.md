# PouchContainer supports LXCFS for highly reliable container isolation

## Introduction

PouchContainer is an open source runtime product of Alibaba. The latest version is 0.3.0 and code address is: [https://github.com/alibaba/pouch](https://github.com/alibaba/pouch). PouchContainer initially designed to support LXCFS to achieve highly reliable container isolation. Linux uses cgroup technology to achieve resource isolation, in the meantime, the host's /proc file system is still mounted in the container. When users read files like /proc/meminfo in the container, it will obtain the information of the host. The lack of `/proc view isolation` in the container can cause series of problems, which will slow down or block the containerization of enterprise business . LXCFS ([https://github.com/lxc/lxcfs](https://github.com/lxc/lxcfs)) is an open source FUSE file system and is use to solve `/proc view isolation` problem and make the container look more like a traditional virtual machine in presentation layer. This article will firstly introduce the applicable business scenarios of LXCFS, secondly, it will briefly introduce the integration of LXCFS within PouchContainer.

## Lxcfs Business Scenario

In the era of physical and virtual machines, our company gradually form our own sets of tool chains, such as compiling and packaging, application deployment, unified monitoring, etc., which provides stable services for applications that are deployed in physical machines and virtual machines. Next, we are going to demonstrate the use of LXCFS in the aspects of monitoring, tools of operation and maintenance and application deployment.

### Monitoring and Operation Tools

Most monitor tools rely on /proc file system to obtain system information. Take Alibaba as an example, some basic monitoring tools of Alibaba collect information through tsar ([https://github.com/alibaba/tsar](https://github.com/alibaba/tsar)),while, tsar relies on /proc file system to collect the information of memory and CPU. We can download source code of tsar and check the use of documents under /proc directory:
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

As we can see, tsar relies on the /proc file system to monitor processes, IO, and CPU.

When /proc file system provides host resource information in the container, these kinds of monitors cannot monitor the information in the container. In order to meet business requirements, it is necessary to adapt container monitor, even to develop another monitoring tool independently for in-container monitoring. This kind of changes will definitely slow down or even block the development of existing business containerization. The container technology should be compatible with original tool chains of company as much as possible and taking the habits of engineers into consideration.

PouchContainer supports LXCFS to solve above problems, relaying on monitoring, operation and maintenance tools of /proc file system. It is transparent to the tools deployed in the container or on the host. The existing monitoring and operation tools can be smoothly migrated into container without adaptation or redevelopment and monitor, achieving monitoring and maintains.

Let's take a look at the example and install PouchContainer 0.3.0 in an Ubuntu virtual machine:

```
# uname -a
Linux p4 4.13.0-36-generic #40~16.04.1-Ubuntu SMP Fri Feb 16 23:25:58 UTC 2018 x86_64 x86_64 x86_64 GNU/Linux
```

Systemd pulls up pouchd , and LXCFS is not enabled by default. The container created cannot use the function of LXCFS at this time . Let's take a look at the contents of the relevant /proc file in the container:

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

We can see, the output of /proc/meminfo and uptime files is the same as the host. The memory is assigned to be 50M as the container starts, the /proc/meminfo file does not reflect the memory limitation in the container.

Launching LXCFS services in host machine and the pouchd process and assigning LXCFS arguments.
```
# systemctl start lxcfs
# pouchd -D --enable-lxcfs --lxcfs /usr/bin/lxcfs >/tmp/1 2>&1 &
[1] 32707
# ps -ef |grep lxcfs
root       698     1  0 11:08 ?        00:00:00 /usr/bin/lxcfs /var/lib/lxcfs/
root       724 32144  0 11:08 pts/22   00:00:00 grep --color=auto lxcfs
root     32707 32144  0 11:05 pts/22   00:00:00 pouchd -D --enable-lxcfs --lxcfs /usr/bin/lxcfs
```

Boot container and obtain relative file contents.
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

use LXCFS to boot container and read /proc file to obtain relevant information in container.

### Business Applications

As to the applications that have strong system reliance, initiator needs obtain memory, CPU and other relative information to set up. 
If /proc fle cannot reflect the resouces condition of container, it will have hug impact on above applications.

For example, for some Java applications, there is also a startup script that looks at the stack size of the /proc/meminfo dynamic allocation runner. When the container memory limit is less than the host memory, a program startup failure occurs caused by a memory allocation failure. For DPDK-related applications, the application tool needs to obtain CPU information based on /proc/cpuinfo to get the CPU logic core used by the application to initialize the EAL layer. If the above information cannot be accurately obtained in the container, the DPDK application needs to modify the corresponding tool.

## PouchContainer Integrates LXCFS

PouchContainer supports LXCFS from version 0.1.0. For details, see: [https://github.com/alibaba/pouch/pull/502] (https://github.com/alibaba/pouch/pull/502) .

In short, when the container starts, mount the LXCFS mount point /var/lib/lxc/lxcfs/proc/ on the host to the virtual /proc filesystem directory inside the container via -v. At this point you can see in the /proc directory of the container, some column proc files, including meminfo, uptime, swaps, stat, diskstats, cpuinfo and so on. The specific use parameters are as follows:

```
-v /var/lib/lxc/:/var/lib/lxc/:shared
-v /var/lib/lxc/lxcfs/proc/uptime:/proc/uptime 
-v /var/lib/lxc/lxcfs/proc/swaps:/proc/swaps 
-v /var/lib/lxc/lxcfs/proc/stat:/proc/stat 
-v /var/lib/lxc/lxcfs/proc/diskstats:/proc/diskstats 
-v /var/lib/lxc/lxcfs/proc/meminfo:/proc/meminfo 
-v /var/lib/lxc/lxcfs/proc/cpuinfo:/proc/cpuinfo
```

To simplify use, the pouch create and run command lines provide the parameter `--enableLxcfs`. Specifying the above parameters when creating the container can omits the complicated `-v` parameter.

After a period of use and testing, we found that the restart of lxcfs, the proc and cgroup will be rebuilt, resulting in a `connect failed` error in accessing /proc in the container. To enhance LXCFS stability, in PR:[https://github.com/alibaba/pouch/pull/885] (https://github.com/alibaba/pouch/pull/885),the guarantee method of refine LXCFS is change to systemd. The specific implementation method is to remount the lxcfs.service plus ExecStartPost, and traverse the container using LXCFS and remount it in the container.

## Summary
PouchContainer supports LXCFS to implement view isolation of the /proc file system in the container, which will greatly reduce the changes of the original tool chain and operation and maintenance habits in the process of containerizing the enterprise inventory, and speed up the containerization process. Strong support for the smooth transition of enterprises from traditional virtualization to container virtualization.
