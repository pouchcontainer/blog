# 使用runV的PouchContainer

近来容器得到了迅速的发展。它为应用程序打包和资源利用改进提供了极大的便利。基于LXC的容器技术带来便利的同时，也损伤了相应的安全性。具体来说，容器共享一台机器的操作系统内核。一旦一个容器试图攻击内核，该主机的所有所有工作负载都将受到影响。

在一些高敏感性和高安全性要求的场景下，纯容器技术存在明显缺陷。更重要的是，在云时代，多租户是云用户的刚性要求。因此，强隔离必须得到保障。换句话讲，容器技术需要更多的安全加固。

[runV](https://github.com/hyperhq/runv) 是 [OCI](https://github.com/opencontainers/runtime-spec)基于管理程序的运行时。通过虚拟化以清晰边界隔离容器和主机的客户内核，runV可以轻松地帮助主机和容器提升安全性。虽然基于管理程序的容器为安全性带来了更多承诺，但它也因虚拟化技术失去了一些性能。且由于需完成更多工作（例如initrd.img加载、内核加载、系统进程启动），启动容器花费的时间随之增长。

## 架构

支持基于管理程序的OCI运行是PouchContainer的目标之一。PouchContainer允许用户决定要创建哪种容器。因此，通过统一输入PouchContainer的API，用户可以创建基于管理程序的容器和基于LXC的容器。使用以上两种运营商时，用户的应用程序可以根据需要灵活选择运行时。
以下是PouchContainer支持runV和runC的架构：

![pouch_with_runv_architecture](https://cdn.nlark.com/yuque/0/2018/png/153684/1532963146910-c872e457-7684-49be-9fd3-53ef6c12c421.png)

## 安装预备

在安装之前，需要提醒一件重要的事情： **带有runV的PouchContainer只能在物理机上运行**. 目前尚不支持嵌套的VM。 此外，我们应该确保在[INSTALLATION.md](https://github.com/alibaba/pouch/blob/master/INSTALLATION.md)中描述的物理机上已经安装了 `containerd` and `pouchd` 。

确保已经完成上述工作。在体验基于管理程序的容器之前，还需要安装其他三个先决条件：

* [QEMU](https://www.qemu.org): 通用机器模拟器和虚拟器。
* [runv](https://github.com/hyperhq/runv): OCI兼容的二进制运行时。
* [hyperstart](https://github.com/hyperhq/hyperstart): 一种基于管理程序容器的小型init服务。

### 安装QEMU

需要[QEMU](https://www.qemu.org) 以运行VM。通过执行以下命令我们可以轻松地安装QEMU相关的工具。

在安装了Ubuntu OS的物理机上：

```
sudo apt-get install -y qemu qemu-kvm
```

在安装了RedHat系列OS的物理机上：

```
sudo yum install -y qemu qemu-kvm
```

### 安装runV

[runv](https://github.com/hyperhq/runv) 不提供二进制包。我们需要通过源代码构建它。

首先，从GitHub克隆runV项目的v1.0.0版本：

```
mkdir -p $Home/go/src/github.com/hyperhq
cd $Home/go/src/github.com/hyperhq
git clone --branch v1.0.0 https://github.com/hyperhq/runv.git
export GOPATH=$HOME/go
```

其次，从源代码构建runV：

```
sudo apt-get install autotools-dev
sudo apt-get install automake
cd runv
./autogen.sh
./configure
sudo make
sudo make install
```

接着二进制的runV将位于您的路径中。

### 安装hyperstart

[hyperstart](https://github.com/hyperhq/hyperstart) 为基于管理程序的容器提供init任务。我们同样需要从源代码版本v1.0.0来构建客内核和initrd.img：

```
cd $Home/go/src/github.com/hyperhq
git clone --branch v1.0.0 https://github.com/hyperhq/hyperstart.git
cd hyperstart
./autogen.sh
./configure
sudo make
```

在成功构建内核和initrd.img之后，我们应该将 `内核` and `initrd.img` 复制到runV将查找的默认目录下：

```
mkdir /var/lib/hyper/
cp build/{kernel,hyper-initrd.img} /var/lib/hyper/
```

## 启动基于管理程序的容器

在安装runV相关工具后，我们需要启动pouchd。然后我们可以通过命令行工具创建基于管理程序的`容器`。创建的容器具有与主机隔离的独立内核。

我们可以通过在命令创建中添加一个标志 `--运行时` 来创建基于管理程序的容器来列举容器。我们还可以使用 `pouch ps` 来列举运行时类型为`runv` 的基于管理程序的容器和基于runc的运行时类型为 `runc`的containerd .

``` shell
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

更重要的是，我们能够检查物理主机和基于管理程序容器的不同内核，如下所示：

``` shell
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

事实证明，在以上实验中，物理主机的内核是4.4.0-101-generic，而基于管理程序的容器的内核是4.12.4-hyper。显然，它们在内核方面是互相隔离的。

## 运行原内核

runV（现在是kataconatiners）提供一种通用方法以提供基于OCI兼容映像的隔离Linux内核。坦白讲，由runV提供的在Guest OS中运行的Linux内核是非常先进和新颖的。但是如何在Guest OS中运行原的Libux内核仍然是业界面临的巨大挑战。

### 场景

使runV提供运行传统Linux内核的Guest OS（如Linux内核2.6.32）是非常合理的。对于整个行业，有太多负载在传统的Linux内核上运行。如果所有这些负载只能在原内核上运行（通常企业中的运营团队无法承担将旧Linux内核升级到4.12或4.14等新内核的风险），那么这样环境下的应用程序无法利用容器和Kubernetes等尖端技术的优势。事实上，毫无疑问，无法获得应用程序的发布速度在互联网时代将是致命的。

### 原内核支持

然而，PouchContainer提出了一种方法来容纳必须在原内核中运行的应用程序。这种解决方法仍在使用runV。但是这中方法需要做很多工作以将新的内核功能向后端移植到原内核中（此部分某种程度上不易开源）。一些演示如何在runV中提供的客户操作系统中运行Linuxneihe 2.6.32：[make legacy kernel in Guest OS](https://www.youtube.com/watch?v=1w5Ams2k-40).

## 结论
PouchContainer带来一种提供基于管理程序容器的普遍方式。通过PouchContainer，用户可以根据特定方案利用基于管理程序容器和基于LXC容器的优势。
