# 介绍
PouchContainer 是高效的、轻量级企业级容器引擎技术，拥有隔离性强、可移植性高、资源占用少等特性。它目前仅支持Linux操作系统，故需要使用虚拟机进行本地运行和测试。本篇指南将指导读者在Mac或Windows上搭建基于VirtualBox + Ubuntu16.04的PouchContainer环境。

# VirtualBox的安装
__1、可以从以下方式中选择其一进行安装：__
* 阿里郎-管家-办公软件管理中搜索VirtualBox，下载并安装。
* 从[https://download.virtualbox.org/virtualbox/5.2.16/VirtualBox-5.2.16-123759-OSX.dmg](https://download.virtualbox.org/virtualbox/5.2.16/VirtualBox-5.2.16-123759-OSX.dmg) 下载安装包，下载并安装

__2、下载完成后，双击VirtualBox.pkg，然后点击“继续”进入安装__

![image.png | left | 506x337](https://cdn.nlark.com/lark/0/2018/png/135654/1532951526444-e74b8693-7e36-40f9-b1da-84d4658d0389.png "")

__3、自定义安装__

![image.png | left | 502x334](https://cdn.nlark.com/lark/0/2018/png/135654/1532951637319-dd1a8547-4c4c-4db1-91cd-4fb9ec386d93.png "")

# VirtualBox的配置
1、__打开VirtualBox，选择"新建"，新建虚拟机__

![image.png | left | 236x61](https://cdn.nlark.com/lark/0/2018/png/124199/1532951984471-9dc5e69b-04d6-4ca3-acfe-cb33341445c4.png "")

__2、设置新建虚拟电脑的名称和系统类型__
* 名称-自定义
* 类型-Linux
* 版本-Ubuntu（64-bit）

![image.png | left | 498x449](https://cdn.nlark.com/lark/0/2018/png/124199/1532952147747-22a62b89-a974-4e25-a835-1327ba455f78.png "")

__3、设置内存大小，可以选择默认大小（1024MB），点击“继续”__

![image.png | left | 497x438](https://cdn.nlark.com/lark/0/2018/png/124199/1532952488126-781dcfd5-521e-46cb-88b0-fe55ddc6c2de.png "")

__4、虚拟硬盘选择：__
* 选择“使用已有的虚拟硬盘文件”，点击选择文件（选择ubuntuPouch.vdi，此Ubuntu已安装PouchContainer），点击“创建”。
* 如果没有这样的文件。点击“现在创建虚拟硬盘”，自行配置系统并手动[安装PouchContainer](https://github.com/alibaba/pouch/blob/master/INSTALLATION.md)。

![image.png | left | 493x430](https://cdn.nlark.com/lark/0/2018/png/124199/1532965407472-70a22b38-dfbb-4598-97ce-f2188ad7bd5d.png "")

# 启动虚拟机并登录pouch
__1、点击“启动”，输入初始的用户名和密码__
```
用户名：pouch
密 码：123456
```

__2、切换到root权限__
```
sudo -i
```

__3、检查网络__
```
ping www.alibaba-inc.com
```

__4、启动pouch服务（默认开机启动）__
```
systemctl start pouch  # 如果有问题，请确认网络连接正确
```

__5、启动一个已经存在的busybox基础容器__
```
pouch run -t -d busybox sh
```

__6、如果您没有busybox，可以pull__
```
pouch pull busybox
```

__7、登录到启动的容器中，其中{ID}是启动命令输出的完整ID中的前六位__
```
pouch exec -it {ID} sh
```
操作成功后如图所示：

![image.png | left | 747x113](https://camo.githubusercontent.com/91f9a4433d2cdbf34b4f3469d4e80c2bd21cfa0a/68747470733a2f2f63646e2e6e6c61726b2e636f6d2f6c61726b2f302f323031382f706e672f3132313937312f313533323335323136373137312d33313637633962362d396234392d343639382d623035362d6532363037336536343331322e706e67 "")

# Mac终端连接到virtualbox虚拟机
使用virtualbox创建虚拟机进行工作，可以有效减少本机环境与工作环境之间的相互影响。但由于Server虚拟机的界面不友好，因而使用SSH连接到虚拟机，使用本地终端编辑是一个非常好的选择。

__1、端口转发__

点击“设置”-“网络”-“高级”-“端口转发”

![image.png | left | 558x389](https://cdn.nlark.com/lark/0/2018/png/135654/1532955458851-bc1c46ed-1ad9-4714-8d4c-1a2deb83f5ec.png "")

点击红框中的icon，设置主机端口**9022**，子系统端口**22**，保存。

![image.png | left | 550x397](https://cdn.nlark.com/lark/0/2018/png/135654/1532955772003-e4dba259-6dae-46bf-96a8-bc9c38f10115.png "")

__2、本机终端连接__

在终端输入
```
$  ssh -p 9022 username@127.0.0.1
```
或者
```
$ ssh -p 9022 username@localhost
```
连接成功效果如图所示：

![image.png | left | 747x352](https://cdn.nlark.com/lark/0/2018/png/135654/1532956235912-f6789f20-c0e8-4758-8f5e-a76fea5b61b7.png "")

至此，您已完成环境搭建，并且可以用终端操作，尽情享受PouchContainer吧！

