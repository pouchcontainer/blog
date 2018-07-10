# 基于 VirtualBox + CentOS7 的 PouchContainer 体验环境搭建与上手指南 for Mac

本篇指南旨在指导基于VirtualBox + CentOS7的PouchContainer环境从零搭建，采用64位CentOS7-Minimal版本。

## 1.安装VirtualBox及虚拟主机

首先在官网[下载](https://www.virtualbox.org/)最新VirtualBox，在系统中安装。随后打开VirtualBox，装载镜像，装载步骤如图：

1.系统类型选择Linux，版本选择Red Hat(64-bit):


<img width="630" height="430" src="https://img.alicdn.com/tfs/TB1quieDxGYBuNjy0FnXXX5lpXa-1578-914.png"/>


2.物理硬盘选择动态分配大小，点击创建:

<img width="630" height="430" src="https://img.alicdn.com/tfs/TB1.KieDxGYBuNjy0FnXXX5lpXa-1492-856.png"/>

3.安装完毕，在VirtualBox中出现创建的虚拟机。右键单击它并选择打开：

<img width="630" height="430" src="https://img.alicdn.com/tfs/TB1.KieDxGYBuNjy0FnXXX5lpXa-1492-856.png"/>

4.进入CentOS安装界面，随后弹出可视化安装界面，注意设置密码：

<img width="630" height="430" src="https://img.alicdn.com/tfs/TB17axjDpOWBuNjy0FiXXXFxVXa-2038-1568.png"/>

5.首次安装完成并重启后，输入在安装时设定的密码进入CentOS命令行，执行命令：
``` bash
ip ad
```
由于新装的系统没有ip地址，此时ip地址如图所示：

<img width="630" height="430" src="https://img.alicdn.com/tfs/TB148NkDuSSBuNjy0FlXXbBpVXa-1440-886.png"/>

没有ip地址就无法进行后续的操作，所以此时首先要开启动态ip
``` bash
cd /etc/sysconfig/network-scripts/
```
<img width="630" height="430" src="https://img.alicdn.com/tfs/TB1F1ieDxGYBuNjy0FnXXX5lpXa-1440-886.png"/>

不同的系统配置文件放置位置可能存在不同，此处配置文件名为`ifcfg-enp0s3`，有些系统中可能为`ifcfg-eth0`，`ifcfg-etchs33`，使用`vi`命令编辑配置文件，修改文件后请记得使用`wq`命令保存并退出：
``` bash
vi ifcfg-enp0s3
```
<img width="630" height="430" src="https://img.alicdn.com/tfs/TB1V7aODv1TBuNjy0FjXXajyXXa-1440-886.png"/>

将`ONBOOT=no`修改为`ONBOOT=yes`后，执行命令重启网络：
``` bash
service network restart
```
至此环境ip配置完毕，可以使用`yum`指令进行后续环境配置。
## 2.在虚拟机环境中安装PouchContainer
PouchContainer相关的rpm包已经放在了阿里云镜像上，您可以将目录配置到`yum`的配置文件中，以方便快速下载安装。在安装pouch前，首先需要安装`yum-utils`以更新`yum-config-manager`：
``` bash
sudo yum install -y yum-utils
```
随后请配置更新PouchContainer的目录：
``` bash
sudo yum-config-manager --add-repo http://mirrors.aliyun.com/opsx/opsx-centos7.repo
sudo yum update
```
上述指令执行成功后，安装PouchContainer的准备工作就全部完成了，执行安装命令：
``` bash
sudo yum install pouch
```
该命令自动安装最新版本PouchContainer，在首次安装时您会收到接受`GPG key`的提示，`key`的秘钥指纹会显示供您参考。
## 3.在虚拟机环境中开启一个PouchContainer的实例
本小节主要内容为在一个新安装好的虚拟机环境中开启一个PouchContainer的实例，供验证环境是否安装成功。首先执行命令启动PouchContainer：
``` bash
sudo systemctl start pouch
```
启动PouchContainer后还需要加载一个镜像文件以启动一个PouchContainer实例，可以下载`busybox`镜像用以测试：
``` bash
pouch pull busybox
```
<img width="630" height="430" src="https://img.alicdn.com/tfs/TB1LuHGDDtYBeNjy1XdXXXXyVXa-1440-886.png"/>

下载好busybox镜像后执行命令以启动busybox基础容器：
``` bash
pouch run -t -d busybox sh
```
执行该命令后如图所示：

<img width="630" height="430" src="https://img.alicdn.com/tfs/TB1wuHGDDtYBeNjy1XdXXXXyVXa-1440-886.png"/>

登录该基础容器：
``` bash
pouchexec -it {ID} sh
// ID 为上图中串码的前6位，在本示例中 ID = 67430c
// 指令为 pouchexec -it 67430c sh
```
登录后如图所示：

<img width="630" height="430" src="https://img.alicdn.com/tfs/TB1WyxlDuuSBuNjy1XcXXcYjFXa-1440-886.png"/>

开心的享受您的PouchContainer容器吧 :D
