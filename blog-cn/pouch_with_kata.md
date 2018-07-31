

# PouchContainer with kata

## 介绍
Kata Container整合了Intel® Clear Containers以及Hyper runV的技术特点，在保证containers运行速度的同时也保证了虚拟机的安全性，其核心技术与runV相同。有关虚拟机container的详细信息，可以参阅 [runV doc](https://github.com/alibaba/pouch/blob/master/docs/features/pouch_with_runV.md).

## 安装准备工作
由于kata不提供安装选项，所以我们从[clear container project](https://github.com/clearcontainers) 上总结了一些安装方法。如果需要了解细节，可以参阅[kata-containers](https://github.com/kata-containers/community#users)。


### 安装步骤

1. 安装qemu

在运行[QEMU](https://www.qemu.org)之前，我们需要先运行VM。通过执行以下代码，我们可以很容易地安装QEMU相关的工具包。 

如果宿主\物理机是 Ubuntu OS 系统，执行:

```
sudo apt-get install -y qemu qemu-kvm
```

如果宿主\物理机是 Red Hat series OS 系统，执行:

```
sudo yum install -y qemu qemu-kvm
```

2. 安装guest kernel以及guest image

[kata-containers/osbuilder](https://github.com/kata-containers/osbuilder) 提供了一个工具用于创建guest image,详情可参考[detail steps](https://github.com/kata-containers/osbuilder#usage)。由于这个工具不能用于创建guest kernel，创建guest kernel的详细步骤可参考[clearcontainers/osbuilder](https://github.com/clearcontainers/osbuilder#build-guest-kernel).

3. 安装 kata-runtime

在这一部分，我们需要安装三个可执行文件：[kata-runtime](https://github.com/kata-containers/runtime), [kata-proxy](https://github.com/kata-containers/proxy) 以及 [kata-shim](https://github.com/kata-containers/shim)。   在一个运行的kata container中，kata-runtime会调用kata-proxy 以及 kata-shim。从源码中获取这些可执行文件其实十分简单，只需要先从github中clone下代码，然后输入make命令。下述代码以获取kata runtime为例：

```shell
git clone https://github.com/kata-containers/runtime.git
cd runtime
make
```

### 配置 kata runtime
Kata runtime从配置文件中直接读取配置参数，配置文件的默认路径为
`/etc/kata-containers/configuration.toml`.
生成默认配置文件的命令为:

```shell
git clone https://github.com/kata-containers/runtime.git
cd runtime
make
```

文件会生成在 `cli/config/configuration.toml`，将这个文件复制到默认路径中即可。

```shell
cp cli/config/configuration.toml /etc/kata-containers/configuration.toml
```

需要注意的是，你可能需要修改这个文件，确保所有可执行文件在系统中的路径都正确。

### 启动kata container

当上述所有步骤全部完成, 你就可以使用kata container了。

```shell
$ pouch run -d --runtime=kata-runtime 8ac48589692a top
00d1f38250fc76b5e66e7fa05a41d342d1b48202d24e2dbf06b20a113b2a008c

$ pouch ps
Name     ID       Status         Created         Image                              Runtime
00d1f3   00d1f3   Up 5 seconds   7 seconds ago   docker.io/library/busybox:latest   kata-runtime
```

进入kata container.

```shell
$ pouch exec -it 00d1f3 sh
/ # uname -r
4.9.47-77.container
```
link: https://github.com/alibaba/pouch/blob/master/docs/features/pouch_with_kata.md