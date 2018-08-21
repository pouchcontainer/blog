## 1. PouchContainer简介
PouchContainer是阿里巴巴集团开源的高效、轻量级企业级富容器引擎技术，可以帮助企业快速提升服务器的利用效率。PouchContainer在阿里内部经过多年的使用，已经具有较强的可靠性和稳定性。目前PouchContainer仅支持在Ubuntu和CentOS上运行。下面将介绍如何在Mac中通过VirtualBox和Ubuntu16.04快速搭建PouchContainer的运行环境，帮助用户在其他的操作系统中也可以使用PouchContainer。Windows系统也可以通过这种方式使用PouchContainer。

## 2. VirtualBox安装
1. VitualBox可以在 https://download.virtualbox.org/virtualbox/5.2.16/VirtualBox-5.2.16-123759-OSX.dmg 这个链接中下载。下载完成后打开VirtualBox-5.2.16-123759-OSX.dmg文件，双击VirtualBox.pkg即可安装。如果需要自定义安装可以选择自定义安装。
![v1.png](https://luolivirtualbox1.oss-cn-beijing.aliyuncs.com/v1.png)
 
## 3. VirtualBox中使用Ubuntu16.04
1. 打开安装好的VirtualBox，首先点击标题栏的New按钮，新建一个操作系统，name可以自定义，type选择Linux，Version选择Ubuntu(64-bit)。
![v2.png](https://luolivirtualbox1.oss-cn-beijing.aliyuncs.com/v2.png)
2. 点击Continue按钮进入内存的选择页面。内存选择1024MB，当然也可以根据需要加大内存。
![v3.png](https://luolivirtualbox1.oss-cn-beijing.aliyuncs.com/v3.png)
3. 点击Continue按钮进入硬盘选择页面，选中"Create a virtual hard disk now"。
![v4.png](https://luolivirtualbox1.oss-cn-beijing.aliyuncs.com/v4.png)
4. 点击Create按钮，选择VDI(VirtualBox Disk Image)。
![v5.png](https://luolivirtualbox1.oss-cn-beijing.aliyuncs.com/v5.png)
5. 点击Continue按钮，选择Dynamically allocated，这样虚拟机可以动态分配空间。
![v6.png](https://luolivirtualbox1.oss-cn-beijing.aliyuncs.com/v6.png)
6. 点击Continue按钮，将文件保存在自己选择的目录下，点击Create按钮，一个虚拟机成功。
![v7.png](https://luolivirtualbox1.oss-cn-beijing.aliyuncs.com/v7.png)
7. 进入创建的虚拟机设置页面，在Storage下将自己下载的ISO文件加载进来。
![v8.png](https://luolivirtualbox1.oss-cn-beijing.aliyuncs.com/v8.png)
8. 点击启动虚拟机，Ubuntu首次启动需要根据自己的偏好对系统进行配置(设置语言，用户名，密码等)。

## 4. 安装PouchContainer
1. 打开virtualBox中已经安装好的ubuntu,输入用户名和密码登录。
2. PouchContainer依赖LXCFS来提供强依赖的保证，因此首先需要安装LXCFS，命令如下所示：
```
sudo apt-get install lxcfs
```
![p1.png](https://luolivirtualbox1.oss-cn-beijing.aliyuncs.com/p1.png)
3. 允许“apt”通过HTTPS下载软件，命令如下所示：
```
sudo apt-get install curl apt-transport-https ca-certificates software-properties-common
```
![p2.png](https://luolivirtualbox1.oss-cn-beijing.aliyuncs.com/p2.png)
4. 添加PouchContainer的官方GPG key，命令如下所示：
```
curl -fsSL http://mirrors.aliyun.com/opsx/pouch/linux/debian/opsx@service.alibaba.com.gpg.key | sudo apt-key add -
```
5. 通过搜索密钥的最后8位BE2F475F来验证是否具有F443 EDD0 4A58 7E8B F645 9C40 CF68 F84A BE2F 475F这个密钥。命令如下：
```
apt-key fingerprint BE2F475F
```
![p3.png](https://luolivirtualbox1.oss-cn-beijing.aliyuncs.com/p3.png)
6. 在新的主机上安装PouchContainer时，需要设置默认的镜像存储库。PouchContainer允许将stable库设为默认的库。命令如下：
```
sudo add-apt-repository "deb http://mirrors.aliyun.com/opsx/pouch/linux/debian/ pouch stable"
```
![p4.png](https://luolivirtualbox1.oss-cn-beijing.aliyuncs.com/p4.png)
7. 通过apt-get下载最新的PouchContainer，命令如下：
```
sudo apt-get update
sudo apt-get install pouch
```
![p5.png](https://luolivirtualbox1.oss-cn-beijing.aliyuncs.com/p5.png)
8. 启动PouchContainer，命令如下：
```
sudo service pouch start
```
9. 下载busybox镜像文件，命令如下：
```
pouch pull busybox
```
10. 运行busybox，命令如下：
```
pouch run -t -d busybox sh
```
11. 容器运行成功后会输出这个容器的ID， 根据这个ID进入busybox的容器中，命令如下:
```
pouch exec -it 23f06f sh
```
![p7.png](https://luolivirtualbox1.oss-cn-beijing.aliyuncs.com/p7.png)
12. 这样就可以在容器内部对容器进行交互。交互完成后输入Exit退出容器。

## 5. 注意事项
1. PouchContainer与Docker冲突，安装PouchContainer前需先检查是否有Docker，否则安装会失败。

## 6. 总结
1. 通过上面的教程，我们可以很轻松的在非Linux电脑上体验PouchContainer。