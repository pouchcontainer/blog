# PouchContainer本地开发环境搭建指南
### 安装Ubuntu16.04 (Xenial LTS)
官网下载链接：

https://wiki.ubuntu.com/XenialXerus/ReleaseNotes?_ga=2.66502190.1690246585.1511691893-1975959426.1511691893

安装PouchContainer,需要安装稳定的Ubuntu版本Ubuntu 16.04 (Xenial LTS)，陈旧版本将不支持或不能测试。

PouchContainer和Docker有冲突，所以必须在安装PouchContainer之前卸载Docker.

**准备**

PouchContainer支持LXCFS去提供强大的隔离功能，所以你应该先安装LXCFS，默认情况下LXCFS是授权的。

``` bash
sudo apt-get install lxcfs
```
![Alt text](https://github.com/ProgrammingK/blog/blob/master/image/1.png)

安装工具包去允许apt使用基于HTTPS的库


``` bash
sudo apt-get install curl apt-transport-https ca-certificates software-properties-common
```
![Alt text](https://github.com/ProgrammingK/blog/blob/master/image/2.png)

**1.增加PouchContainer的官方GPG key**

``` bash
curl -fsSL http://mirrors.aliyun.com/opsx/pouch/linux/debian/opsx@service.alibaba.com.gpg.key | sudo apt-key add -
```
![Alt text](https://github.com/ProgrammingK/blog/blob/master/image/3.png)

**2.设置PouchContainer仓库**

在一个新的主机上第一次安装PouchContainer之前，需要设置PouchContainer仓库，我们默认授权`stabel`仓库，你将经常需要`stabel`仓库。增加`test`仓库，在下面的命令行中增加`test`单词在`stable`单词后面。然后就可以从仓库中安装更新PouchContainer.

``` bash
sudo add-apt-repository "deb http://mirrors.aliyun.com/opsx/pouch/linux/debian/ pouch stable"
```
![Alt text](https://github.com/ProgrammingK/blog/blob/master/image/4.png)

**3.安装PouchContainer**
安装最新的PouchContainer版本

``` bash
# update the apt package index
sudo apt-get update
sudo apt-get install pouch
```
![Alt text](https://github.com/ProgrammingK/blog/blob/master/image/5.png)
![Alt text](https://github.com/ProgrammingK/blog/blob/master/image/6.png)

在安装完PouchContainer后，这个`pouch`分组已经被创建了，但是没有用户加入这个分组。

**4.启动PouchContainer**

``` bash
sudo service pouch start
```
![Alt text](https://github.com/ProgrammingK/blog/blob/master/image/7.png)

以后，你就可以拉一个镜像然后运行PouchContainer容器了。


### 在VirtualBox中选择安装
**1. 配置虚拟机**
打开VirtualBox，新建-名称自定义-类型选择【Linux】-版本选择【Ubuntu (64-bit)】

![Alt text](https://github.com/ProgrammingK/blog/blob/master/image/20180723224944.png)

继续-内存选择【1024M】-继续-使用【已有的虚拟硬盘文件】-选择给定的的vdi文件-创建。

![Alt text](https://github.com/ProgrammingK/blog/blob/master/image/20180723225219.png)

**2. 进入虚拟机**
启动新建实例，等待进入到登录阶段，用户名pouch，密码123456，切换到root用户

**3. 检查网络**
ping www.alibaba-inc.com，检查网络是否正常

**4. 启动pouch服务**
执行命令systemctl start pouch启动pouch服务（默认开机启动）

**5. 创建基础容器并登录**
执行pouch run -t -d busybox sh启动一个busybox基础容器
执行pouch exec -it {ID} sh登入启动的容器，其中ID是上条命令输出的完整ID中的前六位,即可进入容器。

![Alt text](https://github.com/ProgrammingK/blog/blob/master/image/20180723225507.png)

***注意*** 
该镜像中的pouch目录是直接clone了alibaba/pouch中的repo，无法直接在该目录中开发提交，此环境为上手体验环境，开发环境配置请看后一节。

### VritualBox设置共享文件夹

**1. 设置共享文件夹**
在VirtualBox工具页面选择：设置=>共享文件夹；选择本地代码文件夹，设置共享文件夹的名称
![Alt text](https://github.com/ProgrammingK/blog/blob/master/image/20180723214620.png)

设置共享文件夹后，重启虚拟机

**2. 挂载共享文件夹到目标目录下**
使用指令：
```shell
sudo mount -t vboxsf myPouch /root/gopath/src/github.com/alibaba/pouch
```
遇到报错："wrong fs type,bad option,bad superblock..." 等问题，请手动安装如下插件：
```shell
sudo apt install nfs-common
sudo apt install cifs-utils
sudo apt install virtualbox-guest-utils
```

安装插件后仍提示异常：“no such devive......” 运行指令：
```shell
modprobe -a vboxguest vboxsf vboxvideo
```
可参考VirtualBox文档：https://wiki.archlinux.org/index.php/VirtualBox_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87)

**3. 测试挂载成功**
在宿主机上的代码文件夹中增加一个代码无害的测试文件，在虚拟集中打开挂载的目标文件夹，在其中看到目标文件，挂载成功。