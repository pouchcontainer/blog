# 在 PouchContainer 中使用 runV
容器技术最近发展迅速，它为应用程序打包和资源的利用改进提供了很多便利。虽然它带来了一些好处，但与此同时，基于 LXC 的容器技术也带来了一定的安全性问题。具体来说，多个容器会在一台机器上共享操作系统内核。一旦一个容器试图攻击内核，该主机上的所有工作负荷都会因此受到影响。
对于一些对安全性敏感且有严格要求的场景，纯容器技术存在明显的缺陷。更重要的是，在云时代，多租户技术已经成为了云客户的刚性需求。因此，容器之间的强隔离性必须要能得到严格的保证。换句话说，容器技术需要得到更多安全性上的提升。

[runV](https://github.com/hyperhq/runv) 是一个基于虚拟机管理程序（[OCI](https://github.com/opencontainers/runtime-spec)）的运行时。它通过虚拟化 guest kernel，将容器和主机隔离开来，使得其边界更加清晰，这种方式很容易就能帮助加强主机和容器的安全性。虽然基于虚拟机管理程序的容器加强了安全性，但它会因虚拟化技术而损失一些性能。这是因为它需要完成更多的工作，比如 initrd.img 的加载、内核的加载、系统进程的启动等，所以它会需要更多的时间来启动容器。


## 架构
PouchContainer 的目标之一是能兼容支持基于管理程序的 OCI 运行时。PouchContainer 允许用户决定要创建哪种容器。因此，通过调用 PouchContainer 的统一 api 接口，用户可以选择创建基于管理程序的容器，或者是基于 LXC 的容器。通过使用上述两种载体，用户的应用程序可以按需灵活选择运行时。

以下是 PouchContainer 支持 runV 和 runC 的架构图：

![pouch_with_runv_architecture](../static_files/pouch_with_runv_architecture.png)

## 安装前的准备
在安装之前，我们应该记住一件重要的事情：**PouchContainer 中的 runV 仅能在物理机上运行**，目前尚不支持在嵌套的虚拟机上运行。此外，我们还应该确保在 [INSTALLATION.md] 和 [../../INSTALLATION.md] 里描述的物理机上，已经安装了 `containerd` 和 `pouchd`。

在体验基于管理程序的容器之前，除了确保完成了上述的事情外，还需要保证预先安装了以下 3 个部分：

* [QEMU](https://www.qemu.org)：通用的机器模拟器和虚拟器；
* [runv](https://github.com/hyperhq/runv)：与 OCI 兼容的运行时二进制文件；
* [hyperstart](https://github.com/hyperhq/hyperstart)：基于管理程序的容器的微型 init 服务。


### 安装 QEMU
运行虚拟机需要安装[QEMU](https://www.qemu.org)。我们可以执行以下命令来轻松安装 QEMU 相关工具。

在安装了 Ubuntu OS 的物理机上，执行：

```
sudo apt-get install -y qemu qemu-kvm
```

在安装了 RedHat 系列 OS 的物理机上，执行：

```
sudo yum install -y qemu qemu-kvm
```

### 安装 runV
[runV](https://github.com/hyperhq/runv) 不提供二进制包，我们需要基于其源代码来创建这个二进制文件。

首先，从 Github 克隆 runv 项目的 v1.0.0 版本：

```
mkdir -p $Home/go/src/github.com/hyperhq
cd $Home/go/src/github.com/hyperhq
git clone --branch v1.0.0 https://github.com/hyperhq/runv.git
export GOPATH=$HOME/go
``` 

其次，基于源代码来创建 runv 的二进制包：

```
sudo apt-get install autotools-dev
sudo apt-get install automake
cd runv
./autogen.sh
./configure
sudo make
sudo make install
```

执行完以上命令，runv 的二进制文件就已经存在于路径中了。


### 安装 hyperstart
[hyperstart](https://github.com/hyperhq/hyperstart) 为基于管理程序的容器提供了 init 任务。我们还需要基于源代码 v1.0.0 来创建 guest kernel 和 initrd.img：

```
cd $Home/go/src/github.com/hyperhq
git clone --branch v1.0.0 https://github.com/hyperhq/hyperstart.git
cd hyperstart
./autogen.sh
./configure
sudo make
```

在成功创建内核和 initrd.img 之后，我们应该把 `guest kernel` 和 `initrd.img` 复制到 runv 将要查找的默认目录中：

```
mkdir /var/lib/hyper/
cp build/{kernel,hyper-initrd.img} /var/lib/hyper/
```

## 启动基于管理程序的容器
安装完 runv 相关工具后，我们需要启动 pouchd。然后我们可以通过命令行工具 `pouch`，来创建基于管理程序的容器。这里创建的容器具有与主机隔离的独立内核。

我们可以通过在 create 命令中添加 `--runtime` 来创建基于管理程序的容器。我们还可以使用 `pouch ps` 来列出所有容器，包括基于管理程序的容器，其运行时类型是 `runv`，以及运行时类型为 `runc` 的基于 runc 的容器。

```shell
$ pouch create --name hypervisor --runtime runv docker.io/library/busybox:latest
container ID: 95c8d52154515e58ab267f3c33ef74ff84c901ad77ab18ee6428a1ffac12400d, name: hypervisor
$
$ pouch ps
Name         ID       Status    Image                              Runtime
hypervisor   95c8d5   created   docker.io/library/busybox:latest   runv
4945c0       4945c0   stopped   docker.io/library/busybox:latest   runc
1dad17       1dad17   stopped   docker.io/library/busybox:latest   runv
fab7ef       fab7ef   created   docker.io/library/busybox:latest   runv
505571       505571   stopped   docker.io/library/busybox:latest   runc
```

此外，我们能够检查主机物理机和基于管理程序的容器的不同内核，命令如下所示：

```shell
# one host physical machine
$ uname -a
Linux www 4.4.0-101-generic #124-Ubuntu SMP Fri Nov 10 18:29:59 UTC 2017 x86_64 x86_64 x86_64 GNU/Linux
$
# gen tunnel into hypervisor-based container
# check inside kernel
$ pouch start hypervisor -i
/ # uname -a
Linux 4.12.4-hyper #18 SMP Mon Sep 4 15:10:13 CST 2017 x86_64 GNU/Linux
```

事实证明，在上面的实验中，主机物理机中的内核是 4.4.0-101-generic，而基于管理程序的容器中的内核是 4.12.4-hyper。显然，它们在内核上是相互隔离的。

## 运行旧版本的内核
runV（现在是 katacontainers）提供了一种通用方法，它可以提供仍基于 OCI 兼容映像的隔离的 linux 内核。事实上，在 runV 提供的 Guest OS 中，运行其上的 linux 内核是非常新且高级的。但是在使用 runV 时，如何在 Guest OS 中运行旧版本的 linux 内核仍然是业界面临的巨大挑战。

### 场景
使 runV 能够提供运行旧版本 linux 内核（例如 Linux 内核 2.6.32）的 Guest OS 是一个合理的需求。对于整个工业界，有相当多的工作负载是运行在旧版本 Linux 内核上的。如果所有这些工作负载只能在旧内核上运行（通常企业中的运营团队无法承担升级旧版本 Linux 内核的风险，例如将旧 linux 内核升级到 4.12 或 4.14），则此环境中的应用程序将无法利用到容器和 Kubernetes 等尖端技术的优势。实际上，毫无疑问的是，不能提升应用交付速度，这在互联网时代是致命的。


### 旧内核支持
然而，PouchContainer 提出了一种解决方案，可以在仍然使用 runV 的基础上，容纳必须在旧版本内核上运行的应用程序。尽管为了实现该功能，需要付出的代价是，需要做很多工作来将新内核的功能向后端移植到旧内核中（这部分的技术比较难开源）。以下 demo 演示了如何在 runV 提供的 Guest OS 中运行 linux 内核 2.6.32：[在 Guest OS 中创建旧内核](https://www.youtube.com/watch?v=1w5Ams2k-40)。


## 结论
PouchContainer 给出了一种常用方法，可以提供基于管理程序的容器。使用 PouchContainer，用户可以根据特定场景，自行选择使用基于管理程序的容器，或是基于 LXC 的容器。

查看原英文文档，请[点击这里](https://github.com/alibaba/pouch/blob/master/docs/features/pouch_with_runV.md)