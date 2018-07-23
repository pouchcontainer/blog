### 安装Ubuntu16.04 (Xenial LTS)
###Install Ubuntu16.04 (Xenial LTS)
Download Link:
官网下载链接：

https://wiki.ubuntu.com/XenialXerus/ReleaseNotes?_ga=2.66502190.1690246585.1511691893-1975959426.1511691893

To install PouchContainer, you need a maintained version of Ubuntu 16.04 (Xenial LTS). Archived versions aren't supported or tested.

安装PouchContainer,需要安装稳定的Ubuntu版本Ubuntu 16.04 (Xenial LTS)，陈旧版本将不支持或不能测试。

PouchContainer is conflict with Docker, so you must uninstall Docker before installing PouchContainer.

PouchContainer和Docker有冲突，所以必须在安装PouchContainer之前卸载Docker.

**Prerequisites**

**准备**

PouchContainer supports LXCFS to provide strong isolation, so you should install LXCFS firstly. By default, LXCFS is enabled.

PouchContainer支持LXCFS去提供强大的隔离功能，所以你应该先安装LXCFS，默认情况下LXCFS是授权的。

``` bash
sudo apt-get install lxcfs
```
[image/1.png]

Install packages to allow 'apt' to use a repository over HTTPS:

安装工具包去允许apt使用基于HTTPS的库


``` bash
sudo apt-get install curl apt-transport-https ca-certificates software-properties-common
```
[image/2.png]

**1. Add PouchContainer's official GPG key**

**1.增加PouchContainer的官方GPG key**

``` bash
curl -fsSL http://mirrors.aliyun.com/opsx/pouch/linux/debian/opsx@service.alibaba.com.gpg.key | sudo apt-key add -
```
[image/3.png]

**2. Set up the PouchContainer repository**

**2.设置PouchContainer仓库**

Before you install PouchContainer for the first time on a new host machine, you need to set up the PouchContainer repository. We enabled `stabel` repository by default, you always need the `stable` repository. To add the `test` repository, add the word `test` after the word `stable` in the command line below. Afterward, you can install and update PouchContainer from the repository.

在一个新的主机上第一次安装PouchContainer之前，需要设置PouchContainer仓库，我们默认授权`stabel`仓库，你将经常需要`stabel`仓库。增加`test`仓库，在下面的命令行中增加`test`单词在`stable`单词后面。然后就可以从仓库中安装更新PouchContainer.

``` bash
sudo add-apt-repository "deb http://mirrors.aliyun.com/opsx/pouch/linux/debian/ pouch stable"
```
[image/4.png]

**3. Install PouchContainer**

**3.安装PouchContainer**

Install the latest version of PouchContainer.

安装最新的PouchContainer版本

``` bash
# update the apt package index
sudo apt-get update
sudo apt-get install pouch
```
[image/5.png]
[image/6.png]

After installing PouchContainer, the `pouch` group is created, but no users are added to the group.

在安装完PouchContainer后，这个`pouch`分组已经被创建了，但是没有用户加入这个分组。

**4. Start PouchContainer**
**4.启动PouchContainer**

``` bash
sudo service pouch start
```
[image/7.png]

Afterwards, you can pull an image and run PouchContainer containers.

以后，你就可以拉一个镜像然后运行PouchContainer容器了。
