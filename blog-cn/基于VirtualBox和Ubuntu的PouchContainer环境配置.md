# 基于VirtualBox和Ubuntu的PouchContainer环境配置（for Mac）

本文主要介绍容器开发者在VirtualBox和Ubuntu基础上的PouchContainer环境配置。文章从两部分来介绍，希望您能顺利掌握。

# PouchContainer下载
在这个部分，我们分步骤介绍从github上下载PouchContainer文件，这个部分默认使用者有个人github账号，并在本机安装完成git（访问[https://www.git-scm.com/download/](https://www.git-scm.com/download/)下载对应git版本并安装。）
## 1. 从github源fork repo
访问[https://github.com/alibaba/pouch](https://github.com/alibaba/pouch)，并登陆个人github，点击右上角'Fork'。<br>

<div align="center">
    <img src="https://github.com/billhcz/blog/blob/1a9ce3711a7e3d89fe34ab73de7fd12f65498137/img/1.2-1.png" width="200px">
</div><br>

## 2 repo下载
### 2.1 通过git命令下载
点击“Clone or download”并复制路径。<br>

<div align="center">
<img src="https://github.com/billhcz/blog/blob/1a9ce3711a7e3d89fe34ab73de7fd12f65498137/img/1.2-2.png" width="400px">
</div>
<br>

在mac下打开terminal，通过cd命令进入到本地的目标文件夹路径，并输入：
``` javascript?linenums
git clone https://github.com/alibaba/pouch.git
```
## 2.2 通过zip打包文件下载
点击Download ZIP，这种方法更加简单。<br>

<div align="center">
<img src="https://github.com/billhcz/blog/blob/1a9ce3711a7e3d89fe34ab73de7fd12f65498137/img/1.2-3.png" width="400px">
</div>
<br>

然后解压到目标文件夹即可。

# 虚拟机和容器配置
## 1. 虚拟机安装配置
下载安装VirtualBox，下载虚拟机备份ubuntuPouch.vdi<br>
打开VirtualBox，新建-名称自定义-类型选择【Linux】-版本选择【Ubuntu (64-bit)】<br><br>
![enter description here](https://github.com/billhcz/blog/blob/1a9ce3711a7e3d89fe34ab73de7fd12f65498137/img/2.0-1.png)<br><br>
继续-内存选择【1024M】<br><br>
![enter description here](https://github.com/billhcz/blog/blob/1a9ce3711a7e3d89fe34ab73de7fd12f65498137/img/2.0-2.png)<br><br>
继续-使用【已有的虚拟硬盘文件】-选择ubuntuPouch.vdi-创建<br><br>
![enter description here](https://github.com/billhcz/blog/blob/1a9ce3711a7e3d89fe34ab73de7fd12f65498137/img/2.0-3.png)<br><br>
启动新建实例，等待进入到登录阶段，登陆并切换到root用户：

``` javascript?linenums
sudo -i
```
## 2. 共享文件夹挂载
配置环境需要将个人repo通过共享文件夹挂至Ubuntu，为此需要首先安装增强工具。
### 2.1 增强工具安装
安装VBoxLinuxAdditions，点击虚拟机菜单【设备】-点击【安装增强功能…】，然后分别输入:

``` javascript?linenums
sudo apt-get install virtualbox-guest-dkms
sudo mount /dev/cdrom /mnt/
cd /mnt
./VBoxLinuxAdditions.run
```
![enter description here](https://github.com/billhcz/blog/blob/1a9ce3711a7e3d89fe34ab73de7fd12f65498137/img/2.2.1-1.png)

![enter description here](https://github.com/billhcz/blog/blob/1a9ce3711a7e3d89fe34ab73de7fd12f65498137/img/2.2.1-2.png)

![enter description here](https://github.com/billhcz/blog/blob/1a9ce3711a7e3d89fe34ab73de7fd12f65498137/img/2.2.1-3.png)

![enter description here](https://github.com/billhcz/blog/blob/1a9ce3711a7e3d89fe34ab73de7fd12f65498137/img/2.2.1-4.png)
<br>

显示“Do you want to continue?”时，输入Y<br><br>
若出现“VirtualBox Guest Additions: modprobe vboxsf failed”，通过reboot重启虚拟机，并按照2.1步骤切换root用户：

``` javascript?linenums
reboot
```

### 2.2 共享文件夹设置和挂载
设置共享文件夹，点击【设备】-选择【共享文件夹】，设置“共享文件夹路径”为个人repo的文件夹，“共享文件夹名称”设置为“share”-【自动挂载】，【固定分配】-【确定】<br><br>
![enter description here](https://github.com/billhcz/blog/blob/1a9ce3711a7e3d89fe34ab73de7fd12f65498137/img/2.1-1.png)<br>
挂载共享文件夹，输入：

``` javascript?linenums
sudo mount -t vboxsf share /root/gopath/src/github.com/alibaba/
```
其中“share”与“共享文件夹名称”保持一致<br><br>
![enter description here](https://github.com/billhcz/blog/blob/1a9ce3711a7e3d89fe34ab73de7fd12f65498137/img/2.2.2-1.png)<br>

## 3. 启动容器
检查网络是否正常：

``` javascript?linenums
ping www.alibaba-inc.com
```
启动pouch服务（默认开机启动）：

``` javascript?linenums
systemctl start pouch
```
![enter description here](https://github.com/billhcz/blog/blob/1a9ce3711a7e3d89fe34ab73de7fd12f65498137/img/2.3-1.png)
<br>
启动一个busybox基础容器：

``` javascript?linenums
pouch run -t -d busybox sh
```
![enter description here](https://github.com/billhcz/blog/blob/1a9ce3711a7e3d89fe34ab73de7fd12f65498137/img/2.3-2.png)
<br>
登入启动的容器，其中ID是上条命令输出的完整ID中的前六位

``` javascript?linenums
pouch exec -it {ID} sh
```
![enter description here](https://github.com/billhcz/blog/blob/1a9ce3711a7e3d89fe34ab73de7fd12f65498137/img/2.3-3.png)
<br>

# 顺利完成
至此，开发配置环境完成。