# PouchContainer环境搭建和体验

基于`macOS High Sierra 10.13.6`为宿主机，`VirtualBox + Ubuntu16.04 `运行PouchContainer的环境搭建和体验

### 1. 安装虚拟机


笔者的宿主系统为macOS，使用Homebrew安装VirtualBox，关于Homebrew请参考：[https://brew.sh/](https://brew.sh/)

```shell
brew cask search virtualbox
```

接着运行VirtualBox
![undefined](https://cdn.nlark.com/lark/0/2018/png/117932/1532345813010-47020fd7-5bd3-4e0c-8342-45a8bb7ea9df.png) 

新建一个虚拟机
![undefined](https://cdn.nlark.com/lark/0/2018/png/117932/1532345846188-a4c14f90-f965-434a-8e78-b006c79fa75f.png) 

选择钉钉群的磁盘即可
![undefined](https://cdn.nlark.com/lark/0/2018/png/117932/1532345864538-dd950de1-f859-4f7f-b508-8ed90c7813a6.png) 

### 2. 运行虚拟机
先在虚拟机中设置一下端口转发，以便宿主机ssh到虚拟机
![undefined](https://cdn.nlark.com/lark/0/2018/png/117932/1532345957338-86584231-8425-4c05-816e-11e3a7ec83b5.png) 
![undefined](https://cdn.nlark.com/lark/0/2018/png/117932/1532345971592-5cd59593-d240-4e22-95e9-f8347f976bc4.png) 

然后运行虚拟机，账户为pouch，密码123456，检查虚拟机实例是否打开22端口，如果是就exit退出实例
在宿主机中输入

```shell
ssh -l pouch -p 1111 127.0.0.1
```
![undefined](https://cdn.nlark.com/lark/0/2018/png/117932/1532346199103-783b7671-9b6a-490c-8ad8-792774381882.png) 
切换到root
```shell
sudo su root
```

### 3. 运行pouch

先创建pouch实例

```shell

pouch run -t -d busybox sh

```

返回一个key，后面有用

```shell

pouch exec -it <key> sh

```
然后我们就进入了pouch的容器
![undefined](https://cdn.nlark.com/lark/0/2018/png/117932/1532346347168-617f5ffe-5e1f-4519-96ae-28fccd05fbf5.png)