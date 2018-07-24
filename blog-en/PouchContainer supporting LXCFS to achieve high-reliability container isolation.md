origin doc link ：https://github.com/pouchcontainer/blog/blob/master/blog-cn/PouchContainer-%E6%94%AF%E6%8C%81-LXCFS-%E5%AE%9E%E7%8E%B0%E9%AB%98%E5%8F%AF%E9%9D%A0%E5%AE%B9%E5%99%A8%E9%9A%94%E7%A6%BB.md

# PouchContainer supporting LXCFS to achieve high-reliability container isolation

## Introduction
PouchContainer is an open-source runtime container software developed by Alibaba. The latest released version is 0.3.0, located at [https://github.com/alibaba/pouch](https://github.com/alibaba/pouch). PouchContainer is designed to support LXCFS to realize highly reliable container separation. While Linux adopted cgroup technology to achieve resource separation, this solution still causes problem. For example, because host machine's file system usually still stays mounting in container, users will obtain host information instead of actual informations when trying to read files in /proc/meminfo/. The lack of `/proc view isolation` will cause a series of problems and then further stalls or obstructs enterprise business containerization. LXCFS ([https://github.com/lxc/lxcfs](https://github.com/lxc/lxcfs)) is an open-source FUSE file system solution for resolving `/proc view isolation` issue, making the container acting more like a traditional virtual machine in the presentation layer. This article will first introduce the appropriate business scenario for LXCFS and then introduce how LXCFS works in PouchContainer. 


## LXCFS Business Scenario
In the age of physical machine and virtual machine, Alibaba developed an internal toolbox including compiling, packing, application deployment, and unified monitoring. These tools have been providing stable services for applications that are deployed in physical machines and virtual machines. Next, we present how LXCFS works in the containerization process from monitoring, operations, and applications deployment aspects in detail. 


### Monitoring and Operational tools
Most monitoring tools rely on the /proc file system to retrieve system information. In the example of Alibaba's monitoring system, part of the infrastructural monitoring tools collect informations through tsar（[https://github.com/alibaba/tsar](https://github.com/alibaba/tsar)). However, collecting memory and CPU information in tsar depends on /proc file system. We can download tsar source code to learn how tsar uses files under /proc. 

```
$ git remote -v
origin https://github.com/alibaba/tsar.git (fetch)
origin https://github.com/alibaba/tsar.git (push)
$ grep -r cpuinfo .
./modules/mod_cpu.c:    if ((ncpufp = fopen("/proc/cpuinfo", "r")) == NULL) {
:tsar letty$ grep -r meminfo .
./include/define.h:#define MEMINFO "/proc/meminfo"
./include/public.h:#define MEMINFO "/proc/meminfo"
./info.md:memory counter is in /proc/meminfo,there are some key elements
./modules/mod_proc.c:    /* read total mem from /proc/meminfo */
./modules/mod_proc.c:    fp = fopen("/proc/meminfo", "r");
./modules/mod_swap.c: * Read swapping statistics from /proc/vmstat & /proc/meminfo.
./modules/mod_swap.c:    /* read /proc/meminfo */
$ grep -r diskstats .
./include/public.h:#define DISKSTATS "/proc/diskstats"
./info.md:IO Counter is :/proc/diskstats, for example:
./modules/mod_io.c:#define IO_FILE "/proc/diskstats"
./modules/mod_io.c:FILE *iofp;                     /* /proc/diskstats*/
./modules/mod_io.c:    handle_error("Can't open /proc/diskstats", !iofp);
```

It is obvious that tsar's monitoring of processes, IO, and CPU relies on /proc file system. 

When the information provided by /proc file system is from host machines, these monitorings cannot monitor the information in the container. To satisfy business demand to appropriate container monitoring, it is even nessary to develop another set of monitoring tools specifically for a container. This issue will, in nature, stall or even obstruct the containerization of existing enterprise business. Therefore, container technology must be compatible with existing monitoring tools to avoid new tools development and accustoms to Engineers' user habits. 

PouchContainer is a tool that supports LXCFS and capable of getting rid of issues listed above. PouchContainer resolves listed issues by transparentizing the monitoring and maintenence tools that depend on /proc file system deployed in the container or on the host. Existing monitoring and maintenence tools will be able to transition into containers to achieve in-container monitoring and operations without appropriating structure or re-developing tools.

Next, let's see an example of installing PouchContainer 0.3.0 in Ubuntu:



```
# uname -a
Linux p4 4.13.0-36-generic #40~16.04.1-Ubuntu SMP Fri Feb 16 23:25:58 UTC 2018 x86_64 x86_64 x86_64 GNU/Linux
```
systemd invoke pouchd in default mode that does not start LXCFS. Now, the created container cannot use LXCFS functionalities. Let's see the contents under /proc in container:

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
It is obvious to see the consistency between the outputs from /proc/meminfo、uptime files and those from the host machine. Although we designated 50 M memory in start time, /proc/meminfo files does not demonstrate the memory limit in the container. 

Starting LXCFS service inside the host machine, manually invoking pouchd process and designating related relative LXCFS parameters:


```
# systemctl start lxcfs
# pouchd -D --enable-lxcfs --lxcfs /usr/bin/lxcfs >/tmp/1 2>&1 &
[1] 32707
# ps -ef |grep lxcfs
root       698     1  0 11:08 ?        00:00:00 /usr/bin/lxcfs /var/lib/lxcfs/
root       724 32144  0 11:08 pts/22   00:00:00 grep --color=auto lxcfs
root     32707 32144  0 11:05 pts/22   00:00:00 pouchd -D --enable-lxcfs --lxcfs /usr/bin/lxcfs
```

Start the container and get the corresponding file content:

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
To obtain relative information in the container, use containers started by LXCFS and read in-container /proc files 

### Business Applications
For most applications that heavily rely on the operation system, the launch procedure of applications need to obtain information about the system's memory, CPU, and so on.
When the '/proc' file in the container does not accurately reflect the resources condition of the container, it will cause significant effects to above applications.

For example, when some Java applications dynamically allocate the stack size of the running program by checking /proc/meminfo in launch script. When the container memory limit is less than the host memory, programs will fail to launch due to failed memory allocation.

For DPDK related applications, the application tools need to get CPU information and the CPU logic core used by initialization in the EAL layer from /proc/cpuinfo. 
If the above information cannot be accurately obtained in the container, the DPDK application needs to modify the corresponding tools.


## PouchContainer integrated LXCFS
PouchContainer supports LXCFS from version 0.1.0, please check this instance:[https://github.com/alibaba/pouch/pull/502](https://github.com/alibaba/pouch/pull/502) 

In short, when the container starts, it will mount the mount point of LXCFS on the host  /var/lib/lxc/lxcfs/proc/ through -v to the virtual filesystem directory /proc inside the container. 

At this point you can see in the /proc directory of the container, some proc files include meminfo, uptime, swaps, stat, diskstats, cpuinfo and so on.
The parameters are as follows:


```
-v /var/lib/lxc/:/var/lib/lxc/:shared
-v /var/lib/lxc/lxcfs/proc/uptime:/proc/uptime 
-v /var/lib/lxc/lxcfs/proc/swaps:/proc/swaps 
-v /var/lib/lxc/lxcfs/proc/stat:/proc/stat 
-v /var/lib/lxc/lxcfs/proc/diskstats:/proc/diskstats 
-v /var/lib/lxc/lxcfs/proc/meminfo:/proc/meminfo 
-v /var/lib/lxc/lxcfs/proc/cpuinfo:/proc/cpuinfo
```

To simplify usage, the pouch creates and runs command lines to provide parameters  `--enableLxcfs`. If the above parameters are specified when you are creating the container, you can omit the complicated `-v` parameters.

After a period of using and testing, we found after lxcfs restarts, proc and cgroup will be rebuilt. That will cause a `connect failed` error when users access /proc in the container.

In order to enhance the stability of LXCFS, in pull request:[https://github.com/alibaba/pouch/pull/885](https://github.com/alibaba/pouch/pull/885) , the management method of refining LXCFS is now guaranteed by systemd. In order to achieve this, use remote operation by adding ExecStartPost to lxcfs.service, traverse LXCFS container, and then mount again in the container.

## Summary

PouchContainer supports using LXCFS to implement view isolation for /proc filesystems within containers. This will reduce the original tool chains as well as operation and maintenance habits in the containerization process of enterprise storage applications, and speed up the process itself. PouchContainer will provide strong support to enterprises for the smooth transition from traditional virtualization to container virtualization.


