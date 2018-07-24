[PouchContainer](https://github.com/alibaba/pouch) 是阿里巴巴集团开源的高效、企业级容器引擎技术，拥有隔离性强、可移植性高、资源占用少等特点。可以帮助企业快速实现存量业务容器化，同时提高超大规模下数据中心的物理资源利用率。

资源管理是容器运行时的一个重要的部分，本文将给大家介绍PouchContainer资源管理的常用接口和其对应的底层内核接口，为了让读者加深理解，本文为部分接口提供了测试用例。

## 1. PouchContainer资源管理常用接口

| 接口 | 描述 |
| :--- | :--- |
| --blkio-weight | 块设备IO相对权重，取值范围为0到100之间的整数。 |
| --blkio-weight-device | 指定的块设备的IO相对权重 |
| --cpu-period | 完全公平算法中的period值 |
| --cpu-quota | 完全公平算法中的quota值 |
| --cpu-share | CPU份额 (相对权重) |
| --cpuset-cpus | 限制容器使用的cpu核 |
| --cpuset-mems | 限制容器使用的内存节点，该限制仅仅在NUMA系统中生效。 |
| --device-read-bps | 限制对某个设备的读取速率 ，数字需要使用正整数，单位是kb, mb, or gb中的一个。 |
| --device-read-iops | 限制对某个设备每秒IO的读取速率，数字需要使用正整数。 |
| --device-write-bps | 限制对某个设备的写速率 ，数字需要使用正整数，单位是kb, mb, or gb中的一个。 |
| --device-write-iops | 限制对某个设备每秒IO的写速率，数字需要使用正整数。 |
| -m, --memory | 内存使用限制。 数字需要使用整数，对应的单位是b, k, m, g中的一个。 |
| --memory-swap | 总内存使用限制 (物理内存 + 交换分区，数字需要使用整数，对应的单位是b, k, m, g中的一个。 |
| --memory-swappiness | 调节容器内存使用交换分区的选项，取值为0和100之间的整数(含0和100)。 |
| --memory-wmark-ratio | `用于计算low_wmark，取值范围：0到100之间的整数（包含0和100）。` |
| --oom-kill-disable | 内存耗尽时是否杀掉容器 |
| --oom-score-adj | 设置容器进程触发OOM的可能性，值越大时越容易触发容器进程的OOM。 |
| --pids-limit | 用于限制容器内部的pid数量。 |

## 2. PouchContainer资源管理底层的内核技术

### 2.1 Memory资管管理

| 接口 | 对应的内核接口 | 内核接口描述 |
| :--- | :--- | :--- |
| -m, --memory | `cgroup/memory/memory.limit_in_bytes` | 设定内存上限，单位是字节，也可以使用k/K、m/M或者g/G表示要设置数值的单位。 |
| --memory-swap | `cgroup/memory/memory.memsw.limit_in_bytes` | 设定内存加上交换分区的使用总量。通过设置这个值，可以防止进程把交换分区用光。 |
| --memory-swappiness | cgroup/memory/memory.swappiness | 控制内核使用交换分区的倾向。取值范围是0至100之间的整数（包含0和100）。值越小，越倾向使用物理内存。 |
| --memory-wmark-ratio | `cgroup/memory/memory.wmark_ratio` | `用于计算low_wmark，low_wmark = memory.limit_in_bytes * MemoryWmarkRatio。当memory.usage_in_bytes大于low_wmark时，触发内核线程进行内存回收，当回收到memory.usage_in_bytes小于high_wmark时停止回收。` |
| --oom-kill-disable | `cgroup/memory/memory.oom_control` | 如果设置为0，那么在内存使用量超过上限时，系统不会杀死进程，而是阻塞进程直到有内存被释放可供使用时，另一方面，系统会向用户态发送事件通知，用户态的监控程序可以根据该事件来做相应的处理，例如提高内存上限等。 |
| --oom-score-adj | `/proc/$pid/oom_score_adj` | 设置进程触发OOM的可能性，值越大时越容易触发容器进程的OOM。 |

### 2.2 cpu资管管理
| 接口 | 对应的cgroup接口 | cgroup接口描述 |
| :--- | :--- | :--- |
| --cpu-period | `cgroup/cpu/cpu.cfs_period_us` | 负责CPU带宽限制，需要与`cpu.cfs_quota_us`搭配使用。我们可以将period设置为1秒，将quota设置为0.5秒，那么cgroup中的进程在1秒内最多只能运行0.5秒，然后就会被强制睡眠，直到下一个1秒才能继续运行。 |
| --cpu-quota | `cgroup/cpu/cpu.cfs_quota_us` | 负责CPU带宽限制，需要与`cpu.cfs_period_us`搭配使用。 |
| --cpu-share | cgroup/cpu/cpu.shares | 负责CPU比重分配的接口。假设我们在cgroupfs的根目录下创建了两个cgroup（C1和C2），并且将cpu.shares分别配置为512和1024，那么当C1和C2争用CPU时，C2将会比C1得到多一倍的CPU占用率。要注意的是，只有当它们争用CPU时CPU share才会起作用，如果C2是空闲的，那么C1可以得到全部的CPU资源。 |
| --cpuset-cpus | cgroup/cpuset/cpuset.cpus | 允许进程使用的CPU列表（例如：0-4,9）。	 |
| --cpuset-mems | cgroup/cpuset/cpuset.mems | 允许进程使用的内存节点列表（例如：0-1）。 |

### 2.3 io资管管理
| 接口 | 对应的cgroup接口 | cgroup接口描述 |
| :--- | :--- | :--- |
| --blkio-weight | cgroup/blkio/blkio.weight | 设置权重值，取值范围是10至1000之间的整数（包含10和1000）。这跟cpu.shares类似，是比重分配，而不是绝对带宽的限制，因此只有当不同的cgroup在争用同一个块设备的带宽时，才会起作用。 |
| --blkio-weight-device | cgroup/blkio/blkio.weight_device | 对具体的设备设置权重值，这个值会覆盖上述的blkio.weight。 |
| --device-read-bps | `cgroup/blkio/blkio.throttle.read_bps_device` | 对具体的设备，设置每秒读块设备的带宽上限。	|
| --device-write-bps | `cgroup/blkio/blkio.throttle.write_bps_device` | 设置每秒写块设备的带宽上限。同样需要指定设备。	 |
| --device-read-iops | `cgroup/blkio/blkio.throttle.read_iops_device` | 设置每秒读块设备的IO次数的上限。同样需要指定设备。	 |
| --device-write-iops | `cgroup/blkio/blkio.throttle.write_iops_device` | 设置每秒写块设备的IO次数的上限。同样需要指定设备。	 |

### 2.4 其他资源管理接口
| 接口 | 对应的cgroup接口 | cgroup接口描述 |
| :--- | :--- | :--- |
| --pids-limit | cgroup/pids/pids.max | 限制进程数 |

## 3. PouchContainer资源管理接口详解与测试方法
以下内容针对各资源管理接口做了详尽的说明。为了加深读者理解，部分接口提供有测试用例。用例中的PouchContainer版本为0.4.0。如果在你的镜像中stress命令不可用，你可以通过sudo apt-get install stress来安装stress工具。

### 3.1 Memory资管管理
#### 3.1.1 -m, --memory
可以限制容器使用的内存量，对应的cgroup文件是`cgroup/memory/memory.limit_in_bytes`。

单位：b,k,m,g

在默认情况下，容器可以占用无限量的内存，直至主机内存资源耗尽。

运行如下命令来确认容器内存的资源管理对应的cgroup文件。

```
# pouch run -ti --memory 100M reg.docker.alibaba-inc.com/sunyuan/ubuntu:14.04 bash -c "cat /sys/fs/cgroup/memory/memory.limit_in_bytes"
104857600
```

可以看到，当内存限定为100M时，对应的cgroup文件数值为104857600，该数值的单位为字节，即104857600字节等于100M。

本机内存环境为：

```
# free -m
              total        used        free      shared  buff/cache   available
Mem:         257755        2557      254234           1         963      254903
Swap:          2047           0        2047
```

我们使用stress工具来验证内存限定是否生效。stress是一个压力工具，如下命令将要在容器内创建一个进程，在该进程中不断的执行占用内存(malloc)和释放内存(free)的操作。在理论上如果占用的内存少于限定值，容器会工作正常。注意，如果试图使用边界值，即试图在容器中使用stress工具占用100M内存，这个操作通常会失败，因为容器中还有其他进程在运行。

下面尝试对一个限制内存使用为100M的容器执行一个占用150M内存的操作，但容器运行是正常的，没有出现OOM。

```
# pouch run -ti --memory 100M reg.docker.alibaba-inc.com/sunyuan/ubuntu:14.04stress stress --vm 1 --vm-bytes 150M
stress: info: [1] dispatching hogs: 0 cpu, 0 io, 1 vm, 0 hdd
```

通过以下命令看一下系统的内存使用量，你会发现Swap的内存占用量有所增加，说明--memory选项没有限制Swap内存的使用量。

```
#free -m
              total        used        free      shared  buff/cache   available
Mem:         257755        2676      254114           1         965      254783
Swap:          2047          41        2006
```

尝试使用`swapoff -a`命令关闭Swap时我们再次执行早先的命令。从以下log中可以看到，当容器使用内存超过限制时会触发错误。

```
# pouch run -ti --memory 100M reg.docker.alibaba-inc.com/sunyuan/ubuntu:14.04stress stress --vm 1 --vm-bytes 150M
stress: info: [1] dispatching hogs: 0 cpu, 0 io, 1 vm, 0 hdd
stress: FAIL: [1] (422) kill error: No such process
stress: FAIL: [1] (452) failed run completed in 0s
esses
stress: FAIL: [1] (422) kill error: No such process
stress: FAIL: [1] (452) failed run completed in 0s
```

#### 3.1.2 --memory-swap
可以限制容器使用交换分区和内存的总和，对应的cgroup文件是`cgroup/memory/memory.memsw.limit_in_bytes`。

取值范围:大于内存限定值

单位：b,k,m,g

运行如下命令来确认容器交换分区的资源管理对应的cgroup文件。可以看到，当memory-swap限定为1G时，对应的cgroup文件数值为1073741824，该数值的单位为字节，即1073741824B等于1G。

```
# pouch run -ti -m 300M --memory-swap 1G reg.docker.alibaba-inc.com/sunyuan/ubuntu:14.04 bash -c "cat /sys/fs/cgroup/memory/memory.memsw.limit_in_bytes"
1073741824
```

如下所示，当尝试占用的内存数量超过memory-swap值时，容器出现异常。

```
# pouch run -ti -m 100M --memory-swap 200M reg.docker.alibaba-inc.com/sunyuan/ubuntu:14.04stress bash -c "stress --vm 1 --vm-bytes 300M"
stress: info: [1] dispatching hogs: 0 cpu, 0 io, 1 vm, 0 hdd
stress: FAIL: [1] (416) <-- worker 10 got signal 9
stress: WARN: [1] (418) now reaping child worker processes
stress: FAIL: [1] (422) kill error: No such process
stress: FAIL: [1] (452) failed run completed in 0s
```

#### 3.1.3 --memory-swappiness
该接口可以设定容器使用交换分区的趋势，取值范围为0至100的整数（包含0和100）。0表示容器不使用交换分区，100表示容器尽可能多的使用交换分区。对应的cgroup文件是cgroup/memory/memory.swappiness。

```
# pouch run -ti --memory-swappiness=100 reg.docker.alibaba-inc.com/sunyuan/ubuntu:14.04 bash -c 'cat /sys/fs/cgroup/memory/memory.swappiness'
100
```

#### 3.1.4 --memory-wmark-ratio
用于计算low_wmark，`low_wmark = memory.limit_in_bytes * MemoryWmarkRatio`。当`memory.usage_in_bytes`大于low_wmark时，触发内核线程进行内存回收，当回收到`memory.usage_in_bytes`小于high_wmark时停止。对应的cgroup接口是`cgroup/memory/memory.wmark_ratio`。

```
# pouch run -ti --memory-wmark-ratio=60 reg.docker.alibaba-inc.com/sunyuan/ubuntu:14.04 bash -c 'cat /sys/fs/cgroup/memory/memory.wmark_ratio'
60
```
#### 3.1.5 --oom-kill-disable
当out-of-memory (OOM)发生时，系统会默认杀掉容器进程，如果你不想让容器进程被杀掉，可以使用该接口。接口对应的cgroup文件是`cgroup/memory/memory.oom_control`。

当容器试图使用超过限定大小的内存值时，就会触发OOM。此时会有两种情况，第一种情况是当接口--oom-kill-disable=false的时候，容器会被杀掉；第二种情况是当接口--oom-kill-disable=true的时候，容器会被挂起。

以下命令设置了容器的的内存使用限制为20M，将--oom-kill-disable接口的值设置为true。查看该接口对应的cgroup文件，`oom_kill_disable`的值为1。

```
# pouch run -m 20m --oom-kill-disable=true reg.docker.alibaba-inc.com/sunyuan/ubuntu:14.04 bash -c 'cat /sys/fs/cgroup/memory/memory.oom_control'
oom_kill_disable 1
under_oom 0
```

`oom_kill_disable`：取值为0或1，当值为1的时候表示当容器试图使用超出内存限制时（即20M），容器会挂起。

under_oom：取值为0或1，当值为1的时候，OOM已经出现在容器中。

通过`x=a; while true; do x=$x$x$x$x; done`命令来耗尽内存并强制触发OOM，log如下所示。

```
# pouch run -m 20m --oom-kill-disable=false reg.docker.alibaba-inc.com/sunyuan/ubuntu:14.04 bash -c 'x=a; while true; do x=$x$x$x$x; done'

[root@r10d08216.sqa.zmf /root]
#echo $?
137
```

通过上面的log可以看出,当容器的内存耗尽的时候，容器退出，退出码为137。因为容器试图使用超出限定的内存量，系统会触发OOM，容器会被杀掉，此时under_oom的值为1。我们可以通过系统中cgroup文件(`/sys/fs/cgroup/memory/docker/${container_id}/memory.oom_control`)查看`under_oom`的值（`oom_kill_disable` 1，under_oom 1）。

当--oom-kill-disable=true的时候，容器不会被杀掉，而是被系统挂起。

#### 3.1.6 --oom-score-adj
参数--oom-score-adj可以设置容器进程触发OOM的可能性，值越大时越容易触发容器进程的OOM。当值为-1000时，容器进程完全不会触发OOM。该选项对应着底层的`/proc/$pid/oom_score_adj`接口。

```
# pouch run -ti --oom-score-adj=300 reg.docker.alibaba-inc.com/sunyuan/ubuntu:14.04stress bash -c "cat /proc/self/oom_score_adj"
300
```

### 3.2 cpu资源管理
#### 3.2.1 --cpu-period
内核默认的Linux 调度CFS（完全公平调度器）周期为100ms,我们通过--cpu-period来设置容器对CPU的使用周期，同时--cpu-period接口需要和--cpu-quota接口一起来使用。--cpu-quota接口设置了CPU的使用值。CFS(完全公平调度器) 是内核默认使用的调度方式，为运行的进程分配CPU资源。对于多核CPU，根据需要调整--cpu-quota的值。

对应的cgroup文件是`cgroup/cpu/cpu.cfs_period_us`。以下命令创建了一个容器，同时设置了该容器对CPU的使用时间为50000（单位为微秒），并验证了该接口对应的cgroup文件对应的值。

```
# pouch run -ti --cpu-period 50000 reg.docker.alibaba-inc.com/sunyuan/ubuntu:14.04 bash -c "cat /sys/fs/cgroup/cpu/cpu.cfs_period_us"
50000
```

以下命令将--cpu-period的值设置为50000,--cpu-quota的值设置为25000。该容器在运行时可以获取50%的cpu资源。

```
# pouch run -ti --cpu-period=50000 --cpu-quota=25000 reg.docker.alibaba-inc.com/sunyuan/ubuntu:14.04stress stress -c 1
stress: info: [1] dispatching hogs: 1 cpu, 0 io, 0 vm, 0 hdd
```

从log的最后一行中可以看出，该容器的cpu使用率约为50.0%，与预期相符。

```
# top -n1
top - 17:22:40 up 1 day, 57 min,  3 users,  load average: 0.68, 0.16, 0.05
Tasks: 431 total,   2 running, 429 sleeping,   0 stopped,   0 zombie
%Cpu(s):  0.0 us,  0.1 sy,  0.0 ni, 99.9 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
KiB Mem : 26354243+total, 25960588+free,  1697108 used,  2239424 buff/cache
KiB Swap:  2096636 total,        0 free,  2096636 used. 25957392+avail Mem

   PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND
 53256 root      20   0    7324    100      0 R  50.0  0.0   0:12.95 stress
```

#### 3.2.2 --cpu-quota
对应的cgroup文件是`cgroup/cpu/cpu.cfs_quota_us`。

```
# pouch run -ti --cpu-quota 1600 reg.docker.alibaba-inc.com/sunyuan/ubuntu:14.04 bash -c "cat /sys/fs/cgroup/cpu/cpu.cfs_quota_us"
1600
```

--cpu-quota接口设置了CPU的使用值，通常情况下它需要和--cpu-period接口一起来使用。具体使用方法请参考--cpu-period选项。

#### 3.2.3 --cpu-share
对应的cgroup文件是cgroup/cpu/cpu.shares。

```
# pouch run -ti --cpu-quota 1600 reg.docker.alibaba-inc.com/sunyuan/ubuntu:14.04 bash -c "cat /sys/fs/cgroup/cpu/cpu.shares"
1600
```

通过--cpu-shares可以设置容器使用CPU的权重，这个权重设置是针对CPU密集型的进程的。如果某个容器中的进程是空闲状态，那么其它容器就能够使用本该由空闲容器占用的CPU资源。也就是说，只有当两个或多个容器都试图占用整个CPU资源时，--cpu-shares设置才会有效。

我们使用如下命令来创建两个容器，它们的权重分别为1024和512。

```
# pouch run -d --cpuset-cpus=0 --cpu-share 1024 reg.docker.alibaba-inc.com/sunyuan/ubuntu:14.04stress stress -c 1
c7b99f3bc4cf1af94da35025c66913d4b42fa763e7a0905fc72dce66c359c258

[root@r10d08216.sqa.zmf /root]
# pouch run -d --cpuset-cpus=0 --cpu-share 512 reg.docker.alibaba-inc.com/sunyuan/ubuntu:14.04stress stress -c 1
1ade73df0dd9939cc65e05117e3b0950b78079fb36f6cc548eff8b20e8f5ecb9
```

从如下top命令的log可以看到，第一个容器产生的进程PID为10513，CPU占用率为65.1%，第二个容器产生进程PID为10687，CPU占用率为34.9%。两个容器CPU占用率约为2:1的关系，测试结果与预期相符。


```
#top
top - 09:38:24 up 3 min,  2 users,  load average: 1.20, 0.34, 0.12
Tasks: 447 total,   3 running, 444 sleeping,   0 stopped,   0 zombie
%Cpu(s):  3.1 us,  0.0 sy,  0.0 ni, 96.9 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
KiB Mem : 26354243+total, 26187224+free,   964068 used,   706120 buff/cache
KiB Swap:  2096636 total,  2096636 free,        0 used. 26052548+avail Mem

   PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND
 10513 root      20   0    7324    100      0 R  65.1  0.0   0:48.22 stress
 10687 root      20   0    7324     96      0 R  34.9  0.0   0:20.32 stress
```


#### 3.2.4 --cpuset-cpus
该接口对应的cgroup文件是cgroup/cpuset/cpuset.cpus。

在多核CPU的虚拟机中，启动一个容器，设置容器只使用CPU核1，并查看该接口对应的cgroup文件会被修改为1，log如下所示。

```
# pouch run -ti --cpuset-cpus 1 reg.docker.alibaba-inc.com/sunyuan/ubuntu:14.04 bash -c "cat /sys/fs/cgroup/cpuset/cpuset.cpus"
1
```

通过以下命令指定容器使用cpu核1，并通过stress命令加压。

```
# pouch run -ti --cpuset-cpus 1 reg.docker.alibaba-inc.com/sunyuan/ubuntu:14.04 stress -c 1
```

查看CPU资源的top命令的log如下所示。需要注意的是，输入top命令并按回车键后，再按数字键1，终端才能显示每个CPU的状态。

```
#top
top - 17:58:38 up 1 day,  1:33,  3 users,  load average: 0.51, 0.11, 0.04
Tasks: 427 total,   2 running, 425 sleeping,   0 stopped,   0 zombie
%Cpu0  :  0.0 us,  0.3 sy,  0.0 ni, 99.7 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
%Cpu1  :100.0 us,  0.0 sy,  0.0 ni,  0.0 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
%Cpu2  :  0.0 us,  0.0 sy,  0.0 ni,100.0 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
%Cpu3  :  0.0 us,  0.3 sy,  0.0 ni, 99.7 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
```

从以上log得知，只有CPU核1的负载为100%，而其它CPU核处于空闲状态，结果与预期结果相符。

#### 3.2.5 --cpuset-mems
该接口对应的cgroup文件是cgroup/cpuset/cpuset.mems。

```
# pouch run -ti --cpuset-mems=0 reg.docker.alibaba-inc.com/sunyuan/ubuntu:14.04 bash -c "cat /sys/fs/cgroup/cpuset/cpuset.mems"
0
```

以下命令将限制容器进程使用内存节点1、3的内存。

```
# pouch run -ti --cpuset-mems="1,3" reg.docker.alibaba-inc.com/sunyuan/ubuntu:14.04 bash
```

以下命令将限制容器进程使用内存节点0、1、2的内存。

```
# pouch run -ti --cpuset-mems="0-2" reg.docker.alibaba-inc.com/sunyuan/ubuntu:14.04 bash
```


### 3.3 io资管管理

#### 3.3.1 --blkio-weight
通过--blkio-weight接口可以设置容器块设备IO的权重，有效值范围为10至1000的整数(包含10和1000)。默认情况下，所有容器都会得到相同的权重值(500)。对应的cgroup文件为cgroup/blkio/blkio.weight。以下命令设置了容器块设备IO权重为10，在log中可以看到对应的cgroup文件的值为10。

```
# pouch run -ti --rm --blkio-weight 10 reg.docker.alibaba-inc.com/sunyuan/ubuntu:14.04 bash -c "cat /sys/fs/cgroup/blkio/blkio.weight"
10
```

通过以下两个命令来创建不同块设备IO权重值的容器。

```
# pouch run -it --name c1 --blkio-weight 300 reg.docker.alibaba-inc.com/sunyuan/ubuntu:14.04 /bin/bash
# pouch run -it --name c2 --blkio-weight 600 reg.docker.alibaba-inc.com/sunyuan/ubuntu:14.04 /bin/bash
```

如果在两个容器中同时进行块设备操作（例如以下命令）的话，你会发现所花费的时间和容器所拥有的块设备IO权重成反比。

```
# time dd if=/mnt/zerofile of=test.out bs=1M count=1024 oflag=direct
```

#### 3.3.2 --blkio-weight-device
通过--blkio-weight-device="设备名:权重"接口可以设置容器对特定块设备IO的权重，有效值范围为10至1000的整数(包含10和1000)。

对应的cgroup文件为cgroup/blkio/blkio.weight_device。

```
# pouch run --blkio-weight-device "/dev/sda:1000" reg.docker.alibaba-inc.com/sunyuan/ubuntu:14.04 bash -c "cat /sys/fs/cgroup/blkio/blkio.weight_device"
8:0 1000
```

以上log中的"8：0"表示sda的设备号，可以通过stat命令来获取某个设备的设备号。从以下log中可以查看到/dev/sda对应的主设备号为8，次设备号为0。

```
#stat -c %t:%T /dev/sda
8:0
```

如果--blkio-weight-device接口和--blkio-weight接口一起使用，那么Docker会使用--blkio-weight值作为默认的权重值，然后使用--blkio-weight-device值来设定指定设备的权重值，而早先设置的默认权重值将不在这个特定设备中生效。

```
# pouch run --blkio-weight 300 --blkio-weight-device "/dev/sda:500" reg.docker.alibaba-inc.com/sunyuan/ubuntu:14.04 bash -c "cat /sys/fs/cgroup/blkio/blkio.weight_device"
8:0 500
```

通过以上log可以看出，当--blkio-weight接口和--blkio-weight-device接口一起使用的时候，/dev/sda设备的权重值由--blkio-weight-device设定的值来决定。

#### 3.3.3 --device-read-bps
该接口用来限制指定设备的读取速率，单位可以是kb、mb或者gb。对应的cgroup文件是`cgroup/blkio/blkio.throttle.read_bps_device`。

```
# pouch run -it --device /dev/sda:/dev/sda --device-read-bps /dev/sda:1mb reg.docker.alibaba-inc.com/sunyuan/ubuntu:14.04 bash -c "cat /sys/fs/cgroup/blkio/blkio.throttle.read_bps_device"
8:0 1048576
```

以上log中显示8:0 1000,8:0表示/dev/sda, 该接口对应的cgroup文件的值为1048576，是1MB所对应的字节数，即1024的平方。

创建容器时通过--device-read-bps接口设置设备读取速度为500KB/s。从以下log中可以看出,读取速度被限定为498KB/s,与预期结果相符合。

```
# pouch run -it --device /dev/sda:/dev/sda --device-read-bps /dev/sda:500k reg.docker.alibaba-inc.com/sunyuan/ubuntu:14.04 bash
root@r10f10195:/# dd iflag=direct,nonblock if=/dev/sda of=/dev/null bs=5000k coun
1+0 records in
1+0 records out
5120000 bytes (5.1 MB) copied, 10.2738 s, 498 kB/s
```

#### 3.3.4 --device-write-bps
该接口用来限制指定设备的写速率，单位可以是kb、mb或者gb。对应的cgroup文件是`cgroup/blkio/blkio.throttle.write_bps_device`。

```
# pouch run -it --device /dev/sda:/dev/sda --device-write-bps /dev/sda:1mB reg.docker.alibaba-inc.com/sunyuan/ubuntu:14.04 bash -c "cat /sys/fs/cgroup/blkio/blkio.throttle.write_bps_device"
8:0 1048576
```

以上log中显示8:0 1000，8:0表示/dev/sda, 该接口对应的cgroup文件的值为1048576，是1MB所对应的字节数，即1024的平方。

创建容器时通过--device-write-bps接口设置设备写速度为1MB/s。从以下log中可以看出,读取速度被限定为1.0MB/s,与预期结果相符合。

限速操作：

```
# pouch run -it --device /dev/sdb:/dev/sdb --device-write-bps /dev/sdb:1mB reg.docker.alibaba-inc.com/sunyuan/ubuntu:14.04 bash
root@r10d08216:/# dd oflag=direct,nonblock of=/dev/sdb if=/dev/urandom bs=10K count=1000
1024+0 records in
1024+0 records out
10485760 bytes (10 MB) copied, 10.0022 s, 1.0 MB/s
```

#### 3.3.5 --device-read-iops
该接口设置了设备的IO读取速率，对应的cgroup文件是`cgroup/blkio/blkio.throttle.read_iops_device`。

```
# pouch run -it --device /dev/sda:/dev/sda --device-read-iops /dev/sda:400 reg.docker.alibaba-inc.com/sunyuan/ubuntu:14.04 bash -c "cat /sys/fs/cgroup/blkio/blkio.throttle.read_iops_device"
8:0 400
```

可以通过"--device-read-iops /dev/sda:400"来限定sda的IO读取速率(400次/秒)，log如下所示。

```
# pouch run -it --device /dev/sda:/dev/sda --device-read-iops /dev/sda:400 reg.docker.alibaba-inc.com/sunyuan/ubuntu:14.04 bash
root@r10d08216:/# dd iflag=direct,nonblock if=/dev/sda of=/dev/null bs=1k count=1024
1024+0 records in
1024+0 records out
1048576 bytes (1.0 MB) copied, 2.51044 s, 418 kB/s
root@r10d08216:/#
```

通过上面的log信息可以看出，容器每秒IO的读取次数为400，共需要读取1024次（log第二行：count=1024），测试结果显示执行时间为2.51044秒，与预期值2.56(1024/400)秒接近，符合预期。

#### 3.3.6 --device-write-iops
该接口设置了设备的IO写速率，对应的cgroup文件是`cgroup/blkio/blkio.throttle.write_iops_device`。

```
# pouch run -it --device /dev/sda:/dev/sda --device-write-iops /dev/sda:400 reg.docker.alibaba-inc.com/sunyuan/ubuntu:14.04 bash -c "cat /sys/fs/cgroup/blkio/blkio.throttle.write_iops_device"
8:0 400
```

可以通过"--device-write-iops /dev/sda:400"来限定sda的IO写速率(400次/秒)，log如下所示。

```
# pouch run -it --device /dev/sdb:/dev/sdb --device-write-iops /dev/sdb:400 reg.docker.alibaba-inc.com/sunyuan/ubuntu:14.04 bash
root@r10d08216:/# dd oflag=direct,nonblock of=/dev/sdb if=/dev/urandom bs=1K count=1024
1024+0 records in
1024+0 records out
1048576 bytes (1.0 MB) copied, 2.50754 s, 418 kB/s
```

通过上面的log信息可以看出，容器每秒IO的写入次数为400，共需要写1024次（log第二行：count=1024），测试结果显示执行时间为2.50754秒，与预期值2.56(1024/400)秒接近，符合预期。

### 3.4 其他资源管理接口
#### --pids-limit
--pids-limit用于限制容器内部的pid数量，对应的cgroup接口是cgroup/pids/pids.max。

```
# pouch run -ti --pids-limit 100 reg.docker.alibaba-inc.com/sunyuan/ubuntu:14.04stress bash -c "cat /sys/fs/cgroup/pids/pids.max"
100
```

如果在容器内部不断创建新的进程，系统会提示如下错误。

```
bash: fork: retry: Resource temporarily unavailable
bash: fork: retry: Resource temporarily unavailable
```

## 4. 总结
PouchContainer的资源管理依赖于Linux底层的内核技术，感兴趣的读者可以根据自己兴趣补充一些有针对性的测试。关于其依赖的内核技术的实现已经远超本文的范畴。感兴趣的读者可以自行阅读内核手册。

## 参考资料
[PouchContainer社区文档](https://github.com/alibaba/pouch/blob/master/docs/commandline/pouch_run.md)

[Linux Programmer's Manual](http://man7.org/linux/man-pages/man5/proc.5.html)

