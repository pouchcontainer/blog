# 使用 Virutal Box 安装并运行 PouchContainer

本教程将指引你通过 Virtual Box 安装阿里的开源容器 Pouch，并通过 Pouch 运行起 Ubuntu。

## 1. 打开 Virtual Box，并安装 Ubuntu 16.04 的系统

打开 Virtual Box，安装 Ubuntu 16.04 的镜像，启动 Ubuntu 镜像以进行后续安装步骤。

## 2. 安装 PouchContainer

### 0. 先决条件

PouchContainer 使用 LXCFS 来提供强大的个理性，因此你需要首先安装 LXCFS。同时，LXCFS 默认开启:

``` bash
sudo apt-get install lxcfs
```

安装相应的包，来允许 apt 命令使用 HTTPS 协议:

``` bash
sudo apt-get install curl apt-transport-https ca-certificates software-properties-common
```

### 1. 添加 PouchContainer 的官方 GPG 密钥

``` bash
curl -fsSL http://mirrors.aliyun.com/opsx/pouch/linux/debian/opsx@service.alibaba.com.gpg.key | sudo apt-key add -
```

现在，你可以通过搜索指纹的最后 8 位 `F443 EDD0 4A58 7E8B F645  9C40 CF68 F84A BE2F 475F` 来确认密钥是否正确添加。

``` bash
$ apt-key fingerprint BE2F475F
pub   4096R/BE2F475F 2018-02-28
      Key fingerprint = F443 EDD0 4A58 7E8B F645  9C40 CF68 F84A BE2F 475F
uid                  opsx-admin <opsx@service.alibaba.com>
```

### 2. 设置 PouchContainer 的仓库

在新主机上首次安装 PouchContainer 之前，需要先设置PouchContainer存储库。默认启用`stabel`存储库，`stable`存储库是必须要安装的。如果要添加`test`存储库，请在下面的命令行中的`stable`之后添加单词`test`。设置完仓库后，我们可以从存储库安装和更新PouchContainer。

``` bash
sudo add-apt-repository "deb http://mirrors.aliyun.com/opsx/pouch/linux/debian/ pouch stable"
```

### 3. 安装 PouchContainer

安装最新版本的 PouchContainer

``` bash
# update the apt package index
sudo apt-get update
sudo apt-get install pouch
```

**4. 开启 PouchContainer **

``` bash
sudo service pouch start
```

### 5. 测试 PouchContainer 安装

``` bash
sudo pouch pull hello-world
sudo pouch run hello-world
```

如果提示无法连接 registry.hub.docker.com，可以使用国内的镜像网站

``` bash
sudo pouch pull registry.docker-cn.com/library/hello-world
sudo pouch run hello-world
```

## 3. 通过 PouchContainer 来执行 Ubuntu

### 1. 拉取 Ubuntu 的镜像

``` bash
sudo pouch pull ubuntu
```

### 2. 执行 Ubuntu 镜像

以交互模式启动 Ubuntu，并启动 tty:

``` bash
sudo pouch run -it ubuntu
```