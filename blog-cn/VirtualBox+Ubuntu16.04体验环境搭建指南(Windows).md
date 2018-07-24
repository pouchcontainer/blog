# 1. PouchContainer简介
 [PouchContainer](https://github.com/alibaba/pouch)是阿里巴巴集团开源的高效、轻量级企业级富容器引擎技术，它具备强隔离和可移植性等特点，可用来帮助企业快速实现存量业务容器化，以及提高企业内部物理资源的利用率。

PouchContainer 源自阿里巴巴内部场景，诞生初期，在如何为互联网应用保驾护航方面，倾尽了阿里巴巴工程师们的设计心血。PouchContainer 的强隔离、富容器等技术特性是最好的证明。在阿里巴巴的体量规模下，PouchContainer 对业务的支撑得到双 11 史无前例的检验，开源之后，阿里容器成为一项普惠技术，定位于「助力企业快速实现存量业务容器化」。 
# 2. 体验环境搭建
【注】整个第二节为上手体验环境，该镜像中的pouch目录是直接clone了alibaba/pouch中的repo，无法直接在该目录中开发提交！
## 2.1. 安装VirtualBox
1. 打开【阿里郎】->选择左侧栏中的【管家】->办公软件管理，具体如图：
![undefined](https://cdn.nlark.com/lark/0/2018/png/132231/1532344205392-0ba64dae-b407-4ff7-b8b9-f221a93ae728.png)
2. 在搜索框中输入VirtualBox，并安装
![undefined](https://cdn.nlark.com/lark/0/2018/png/132231/1532344290135-ed2e9b67-766e-45ef-a72a-9ff034f27e22.png) 
【注】若没有【阿里郎】可在钉盘上下载：
Mac版本地址：https://space.dingtalk.com/s/gwHOABma4QLOGlgkPQPaACBiMzk5ZWRjZTAyOGI0MTBkOGRkNTRjYzNkN2Q1NTFjOA  密码: p5Sb
Windows版本地址：https://space.dingtalk.com/s/gwHOABmLzwLOGlgkPQPaACBhNzNjYjI5NTYxMzQ0NmUwOWRmMTFlN2UzMTYxNDQ4Mw  密码: V7ms
## 2.2. 下载开发环境的虚拟机备份
群文件中下载`ubuntuPouch.vdi`，此文件大小约3.5GB。
## 2.3. 搭建开发虚拟机环境
1. 在Oracle VM VirtualBox中选择"新建"，虚拟机名称可自定义，类型选择“Linux”，版本选择“Ubuntu (64-bit)”,并选择下一步，具体如图：
![undefined](https://cdn.nlark.com/lark/0/2018/png/132231/1532344993077-0cd56605-8b6f-4dc0-902e-0473ce67b629.png) 
2. 内存选择1024M，具体如图：
![undefined](https://cdn.nlark.com/lark/0/2018/png/132231/1532345103260-6995b132-bbdc-49e5-93fd-cfbe30fb3f79.png) 
3. 选择【已有的虚拟硬盘文件】-选择步骤2.2中下载的`.vdi`文件，并选择“创建”。
![undefined](https://cdn.nlark.com/lark/0/2018/png/132231/1532345138776-73add3b0-9f87-45e0-bf4b-9db6ef7d9b06.png) 
## 2.4. 启动新建实例
1. 等待进入到登录阶段，用户名pouch，密码123456。
![undefined](https://cdn.nlark.com/lark/0/2018/png/132231/1532345382824-733e22eb-234c-4c8f-b22c-f1cd2a7a8553.png) 
2. 命令行输入`sudo su`,切换到root用户,切换成功后会看到用户名会有变化：
![undefined](https://cdn.nlark.com/lark/0/2018/png/132231/1532345631234-fd78bbb1-fafa-4bc5-b972-b688b56eaec6.png) 
## 2.5. 检查网络
命令行中输入命令：`ping alibaba-inc.com`,检查网络是否畅通。
## 2.6. 启动pouch服务
1. 命令行中输入命令：`systemctl start pouch`
2. 命令行中输入命令：`pouch run -t -d busybox sh`启动一个busybox基础容器，之后会显示busybox的ID
3. 命令行中输入命令：`pouch exec -it {ID} sh`登入启动的容器。其中，{ID}是上条命令输出的完整ID中的前六位。
![undefined](https://cdn.nlark.com/lark/0/2018/png/132231/1532346183142-78265ff4-2b39-4947-9221-5bc2642bdd05.png) 
此时，VirtualBox + Ubuntu16.04环境中已能成功运行起PouchContainer。体验环境中已经包含的工具有：vim、make、git、go等基本工具。其中pouch的源码路径位于`/root/gopath/src/github.com/alibaba/pouch`路径下
# 3. 共享文件夹挂载
## 3.1. 下载最新版pouch代码
1. 配置git所需环境，如用户名和邮箱
2. git中输入：`git clone@github.com:Qiaoxinshu/pouch.git`
![undefined](https://cdn.nlark.com/lark/0/2018/png/132231/1532347763099-2c1368c2-ccce-436b-898e-e2f965d0c4e2.png) 
## 3.2. 添加共享文件夹
Oracle VM VirtualBox界面中选择“baiji-5#-demo”【设置】->左侧边栏选择【共享文件夹】->【添加共享文件夹】，共享刚刚拉取的pouch文件夹，并选择“固定分配”。
![undefined](https://cdn.nlark.com/lark/0/2018/png/132231/1532347966375-ae70577f-a34c-41da-afda-b5cf8636da95.png) 
之后会在【固定分配】中看到添加好的固定文件夹
![undefined](https://cdn.nlark.com/lark/0/2018/png/132231/1532348064973-0deaacc6-5e49-4c27-83f7-e2410d4afcef.png) 
## 3.3. 虚拟机挂载
1. 进入到`/root/gopath/src/github.com/alibaba/`目录下
2. 将原有的`pouch`文件夹删除：`sudo rm -rf pouch`
3. 下载VitualBox增强插件：`sudo apt-get install virtualbox-guest-x11`，注意安装后要重启虚拟机
4. 挂载pouch目录：`sudo mount -t vboxsf pouch/ pouch/`，成功后便会在当前目录下看到pouch文件夹。
![undefined](https://cdn.nlark.com/lark/0/2018/png/132231/1532349559864-9fa068fa-05b7-4231-9f36-6dfca1719a5c.png) 
