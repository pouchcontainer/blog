#基于VirtualBox + CentOS7的PouchContainer体验环境搭建与上手指南

###什么是*PouchContainer*？
*PouchContainer*是一款由阿里巴巴开源的具有隔离性强、可移植性高、资源占用少等特性的高效、轻量的企业级富容器引擎技术。

###环境选择
由于*PouchContainer*是企业级容器方案，只支持Linux操作系统，故需要使用虚拟机才能做到本地运行和测试，本文选择*VirtualBox*作为虚拟化平台。而*CentOS(Community Enterprise Operating System)* 是社区企业操作系统基于*Linux*的发行版本之一，本文使用*CentOS7*这个版本，对虚拟机安装操作系统。

##环境搭建步骤

###*VirtualBox*的下载
* 使用阿里郎-管家-办公软件管理 安装*VirtualBox*，默认版本为5.2.12。
* 若没有阿里郎，可在钉盘上下载  

*Mac*版本地址：  
[https://space.dingtalk.com/gwHOABma4QLOGlgkPQPaACBiMzk5ZWRjZTAyOGI0MTBkOGRkNTRjYzNkN2Q1NTFjOA](https://space.dingtalk.com/gwHOABma4QLOGlgkPQPaACBiMzk5ZWRjZTAyOGI0MTBkOGRkNTRjYzNkN2Q1NTFjOA)  密码: p5Sb  
*Windows*版本地址：  
[https://space.dingtalk.com/s/gwHOABmLzwLOGlgkPQPaACBhNzNjYjI5NTYxMzQ0NmUwOWRmMTFlN2UzMTYxNDQ4Mw](https://space.dingtalk.com/s/gwHOABmLzwLOGlgkPQPaACBhNzNjYjI5NTYxMzQ0NmUwOWRmMTFlN2UzMTYxNDQ4Mw)  密码: V7ms

###*CentOS*的下载
*CentOS7-Minimal*的安装镜像钉盘地址： 
[https://space.dingtalk.com/s/gwHOABmslALOGlgkPQPaACAwNTg4YTBjOGI4OTI0MGQ5YjE5MDgyYWFjMzAxMDY1MQ](https://space.dingtalk.com/s/gwHOABmslALOGlgkPQPaACAwNTg4YTBjOGI4OTI0MGQ5YjE5MDgyYWFjMzAxMDY1MQ) 密码: tkD3

##环境安装步骤
###1.*Windows*安装步骤  
第一步,安装包下载完毕，开始安装，点击"下一步"  

![](https://i.imgur.com/JgZy07D.jpg)  

第二步，自定义安装，选择安装功能的方式，默认即可，选择安装位置后即可，点击"下一步"    

![](https://i.imgur.com/j2Xa2zs.jpg)  

第三步，自定安装，默认即可，点击"下一步"  

![](https://i.imgur.com/EVhZFMv.jpg)  

第四步，网络界面，点击"是"  

![](https://i.imgur.com/zMWg5Uc.jpg)  

第五步，安装开始，点击"安装"  

![](https://i.imgur.com/73TH8ZA.jpg)

第六步，点击"完成"，安装完成！  

![](https://i.imgur.com/PN1jAlJ.jpg)  

###2.*Mac*安装步骤   
按照操作步骤基本与*Windows*一致 
##虚拟机新建实例步骤  

第一步，打开"*Oracle VM VirtualBox*"，点击左上角"新建"，创建实例  

![](https://i.imgur.com/axy1O43.jpg)  

第二步，配置新建虚拟电脑，自定义名称，类型选择"*Linux*"，版本:"*Red Hat*(64-bit)"，内存大小1024M，选择"现在创建虚拟硬盘"，点击"创建"  

**注意！！有一些windows系统在版本选择上没有64bit的选项，原因是开机启动项的BIOS设置没有打开虚拟化，做法:重新开机-开机后立刻长按F2(不同电脑可能有所不同)，进入BIOS-找到"Security"选项-选择"Virtualization"-将设置都调为"Enabled"-再次重启后，运行"Oracle VM VirtualBox"即可**  

![](https://i.imgur.com/pKuRT2S.jpg)  

第三步，创建虚拟硬盘，选择文件位置，大小默认即可，虚拟硬件文件类型:"VDI"，"存储在物理硬盘上"选择"动态分配"，点击"创建"，实例创建完成

![](https://i.imgur.com/j3S5Z5T.jpg)  

第四步，选择已创建的实例点击"设置"，配置下载的*CentOS7*镜像文件  

![](https://i.imgur.com/mvEfC09.jpg)  

第五步，选择"存储"，在存储介质中点击控制器:IDE的"没有盘片"，其属性分配光驱，点击右边的光碟图标，添加下载好的*CentOS7*的路径，点击"ok"完成设置

![](https://i.imgur.com/92Pgf24.jpg)

**P.S:
若在钉盘地址  
[https://space.dingtalk.com/s/gwHOABmLxALOGlgkPQPaACA4N2JjNmIwMGI5NWU0MGE1YjZhNTBiOGNjMDZhOTJiNQ](https://space.dingtalk.com/s/gwHOABmLxALOGlgkPQPaACA4N2JjNmIwMGI5NWU0MGE1YjZhNTBiOGNjMDZhOTJiNQ) 密码: pLgb   
下载了虚拟硬盘文件，可直接在上述的第二步，选择"使用已有的虚拟硬盘文件"，然后直接跳过第三、四、五步并结束**

##虚拟机的启动
完成虚拟机的基本设置后，就可以启动虚拟机了，选择已创建的实例，点击"启动"，开启虚拟机后，鼠标单击黑色区域，使用方向上下键选择第一条

![](https://i.imgur.com/2WkLfea.jpg)

等待文件加载，加载完成后，显示如下界面，可选择语言，之后点击"确定"

![](https://i.imgur.com/1HKTQUX.jpg)

出现安装信息摘要，等待图标由灰色变为黑色，"开始安装"亮起后，点击"开始安装"  
   
![](https://i.imgur.com/u4MEYOE.jpg)   

在用户设置中点击"*ROOT*密码"，用户名*root*，设置密码*Ali88Baiji*，可完成"创建用户"，最后点击"完成"

![](https://i.imgur.com/x7hnRdf.jpg) 

完成安装后，点击"重启"，即可开始使用。

#*PouchContainer*的安装
想要在新的虚拟机上安装*PouchContainer*，首先，需要安装*pouch*包以便进行配置，但因为*CentOS*默认是不联网的，所以要先将虚拟机联网，步骤如下:    
启动虚拟机，进入目录 *cd /etc/sysconfig/network-scripts/*

![](https://i.imgur.com/fF4QnmB.jpg)

查看网卡配置文件，将 *ONBOOT=no* 改为*ONBOOT=yes*  

![](https://i.imgur.com/gd9OH5f.jpg)  

至此，虚拟机已经成功连上网，可用ping检验。  
同时，为了之后的一些操作我们可以预安装一些必要的包:  
*yum install automake autoconf  git  pkg-config make gcc golang qemu aclocal libseccomp-devel -y*  
然后，我们就可以进行*Pouch*包的安装:  
*sudo yum install -y yum-utils*  
*sudo yum-config-manager --add-repo http://mirrors.aliyun.com/opsx/opsx-centos7.repo*
*sudo yum update*  
*sudo yum install pouch*  
同时也可以按如下方式:  
*mkdir -p $GOPATH/src/github.com/alibaba/*   
*cd $GOPATH/src/github.com/alibaba/*   
*git clone https://github.com/alibaba/pouch.git*  
至此，*PouchContainer*的安装完成！

##*PouchContainer*的启动
到了这一步，就可以实现我们的最终目标了:运行*PouchContainer*  
*sudo systemctl start pouch*  
若想创建交互式的容器实例，向终端输入如下命令即可实现:  
先pull一个image，以"busybox"为例:  
*pouch pull busybox*  
然后就可以利用:  
*pouch run -i busybox*建立一个交互式容器的实例了!
 




