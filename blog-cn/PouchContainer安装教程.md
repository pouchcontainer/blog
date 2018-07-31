# PouchContainer安装教程中文版

## 环境和前期准备


- 宿主机系统：CentOS 7 / Ubuntu 16.04
- 需要的软件：VirtualBox
- Virtualbox软件包获取方式：
使用阿里郎-管家-办公软件管理安装VirtualBox，默认版本为5.2.12。若没有阿里郎，请在钉盘上下载

Mac版本地址：
[Mac-下载地址](https://space.dingtalk.com/s/gwHOABma4QLOGlgkPQPaACBiMzk5ZWRjZTAyOGI0MTBkOGRkNTRjYzNkN2Q1NTFjOA)
&emsp;&emsp;&emsp;密码:p5Sb

Windows版本地址：
[Windows-下载地址](https://space.dingtalk.com/s/gwHOABmLzwLOGlgkPQPaACBhNzNjYjI5NTYxMzQ0NmUwOWRmMTFlN2UzMTYxNDQ4Mw) 
&emsp;&emsp;&emsp;密码:V7ms

## 加载虚拟机和配置

### 虚拟机加载
1. 在钉钉群中下载镜像，CentOS或Ubuntu都可以
2. 打开Virtualbox，点击新建
3. 虚拟机名称自起，类型选择`Linux`,版本选择`Linux 2.6／3.x/4.x(64-bit)`
4. 建议内存大小，默认`1024`即可
5. 选择`使用已有的虚拟硬盘文件`，选择下载的虚拟机，点击创建
6. 启动自己的虚拟机

### 虚拟机配置

#### CentOS
用户名：root

密  码：Ali88Baiji

尝试ping www.alibaba-inc.com，若能ping通，进入下一步

若不能，则要修改网卡信息，使用 `ip ad` 命令查看Virtualbox分配给虚拟机网卡的mac地址，如下图所示：

![avatar](https://images2018.cnblogs.com/blog/761101/201807/761101-20180731012815511-1807884097.png)

使用下面命令修改当前虚拟机的IP地址：

```
vi /etc/sysconfig/network-scripts/ifcfg-eth0
```
修改 `HWADDR=` 对应的值，修改为自己的查询结果(红框内的结果)。

#### Ubuntu
用户名：pouch

密  码：123456

尝试ping www.alibaba-inc.com，若能ping通，进入下一步

## 安装PouchContainer
### CentOS
#### 1.安装所需的包yum-utils

```
sudo yum install -y yum-utils
``` 
结果图：

![avatar](https://images2018.cnblogs.com/blog/761101/201807/761101-20180731011533827-586132642.png)


#### 2. 添加PouchContainer库

```
sudo yum-config-manager --add-repo http://mirrors.aliyun.com/opsx/opsx-centos7.repo
sudo yum update
``` 
结果图：

![avatar](https://images2018.cnblogs.com/blog/761101/201807/761101-20180731011449985-103277584.png)


#### 3. 安装PouchContainer

```
sudo yum install pouch
``` 
结果图：

![avatar](https://images2018.cnblogs.com/blog/761101/201807/761101-20180731011458334-832954266.png)


#### 4. 运行PouchContainer

```
//设置开机启动
systemctl start pouch
//拉取远程镜像
pouch pull busybox
``` 
结果图：

![avatar](https://images2018.cnblogs.com/blog/761101/201807/761101-20180731011509633-1996626722.png)


```
//启动镜像，前6个字符为登陆需要的ID
pouch run -t -d busybox sh
```
结果图：

![avatar](https://images2018.cnblogs.com/blog/761101/201807/761101-20180731011519379-266427941.png)


```
//登陆镜像，执行一些常见命令
pouch exec -it {ID} sh
```

结果图：

![avatar](https://images2018.cnblogs.com/blog/761101/201807/761101-20180731011527313-674343869.png)

### Ubuntu

#### 1.安装LXCFS

```
sudo apt-get install lxcfs
```

结果图：

![avatar](https://images2018.cnblogs.com/blog/761101/201807/761101-20180731011541046-1004355148.png)

#### 2.安装下列包以允许'apt'通过HTTPS使用仓库

```
sudo apt-get install curl apt-transport-https ca-certificates software-properties-common
```

结果图：

![avatar](https://images2018.cnblogs.com/blog/761101/201807/761101-20180731011649030-1905207872.png)

#### 3.添加PouchContainer的官方GPG密钥

```
curl -fsSL http://mirrors.aliyun.com/opsx/pouch/linux/debian/opsx@service.alibaba.com.gpg.key | sudo apt-key add -
```

结果图：

![avatar](https://images2018.cnblogs.com/blog/761101/201807/761101-20180731011700640-614313500.jpg)


#### 4.通过搜索指纹的最后8个字符，验证您现在是否具有指纹`F443 EDD0 4A58 7E8B F645 9C40 CF68 F84A BE2F 475F`的密钥。

```
apt-key fingerprint BE2F475F
```

结果图：

![avatar](https://images2018.cnblogs.com/blog/761101/201807/761101-20180731011710188-616788657.jpg)

#### 5.建立PouchContainer仓库

```
sudo add-apt-repository "deb http://mirrors.aliyun.com/opsx/pouch/linux/debian/ pouch stable"
```

结果图：

![avatar](https://images2018.cnblogs.com/blog/761101/201807/761101-20180731011718977-2015734432.jpg)


#### 6.安装PouchContainer

```
sudo apt-get update
sudo apt-get install pouch
```

结果图：

![avatar](https://images2018.cnblogs.com/blog/761101/201807/761101-20180731011828447-1398873692.jpg)

#### 7.启动PouchContainer(同CentOS环境)

## shh远程登陆&文件共享

### shh远程登陆

#### 1.配置网卡为桥接模式
virtualbox的NAT只支持虚拟机到宿主机的单向通信，即虚拟机可以ping通宿主机，但宿主机无法ping通虚拟机，故需将网络配置改为桥接模式

![avatar](https://cdn.nlark.com/lark/0/2018/png/123186/1532952006217-7fd4751c-f246-4008-8241-ae85af32ecf0.png)

#### 2.安装ssh服务并开启，默认端口号是22 

```
apt-get install openssh-server
```
![avatar](https://images2018.cnblogs.com/blog/761101/201807/761101-20180731094405012-1522115623.png)

#### 3.查看ssh服务是否开启

![avatar](https://cdn.nlark.com/lark/0/2018/png/123186/1532952211218-fa86e7f1-3701-4cb9-bebd-c19bffed324c.png)

#### 4.获取虚拟机IP

![avatar](https://cdn.nlark.com/lark/0/2018/png/123186/1532952133466-32a66208-df68-47c7-9124-6d9dc8597a7f.png)

#### 5.ssh登陆   
 
```
ssh 用户名@IP
```
![avatar](https://cdn.nlark.com/lark/0/2018/png/123186/1532952276881-e3af58c4-7f23-4f40-81e1-b9a187ec9f5d.png)




### 文件共享

#### 1.配置共享文件夹 virtualbox -> 设置 -> 共享文件夹

![avatar](https://cdn.nlark.com/lark/0/2018/png/123186/1532952361541-99ba9094-7f9c-4a3b-ba7e-9822a2196ad7.png)

#### 2.安装增强功能包：
如果要用VirtualBox自带的共享文件夹功能，必须先安装Guest Additions。

安装方法：置顶的菜单条->devices->Install Guest Additions

![avatar](https://cdn.nlark.com/lark/0/2018/png/123186/1532952462469-cbd198d6-ac62-4565-ae4c-e76e0ebd8355.png)

配置为第一IDE从通道

![avatar](https://cdn.nlark.com/lark/0/2018/png/123186/1532952473708-06dcb074-1358-48d5-bcb1-ddd590461790.png)

#### 3.文件挂载
如果遇到unknown filesystem type "vboxsf"问题，则执行下述命令即可：

```
apt-get update
apt-get install virtualbox-guest-utils
```

![avatar](https://cdn.nlark.com/lark/0/2018/png/123186/1532952524332-240c1604-9e57-464e-8132-fcecdc872906.png)

然后重启虚拟机，重新挂载

![avatar](https://cdn.nlark.com/lark/0/2018/png/123186/1532951986751-fb248032-4172-4a34-87ac-270fef0f75da.png)



