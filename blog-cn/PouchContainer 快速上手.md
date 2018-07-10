# 安装虚拟机环境

由于PouchContainer是企业级容器方案，故其只支持Linux操作系统。Windows和Mac的用户需要在虚拟机环境下运行，所以需要配置相关环境，Linux用户请跳过该环节。

## 安装virtualbox

使用阿里郎-管家-办公软件管理安装VirtualBox，默认版本为5.2.12。若没有阿里郎，请在钉盘上下载

Mac版本地址：
https://space.dingtalk.com/s/gwHOABma4QLOGlgkPQPaACBiMzk5ZWRjZTAyOGI0MTBkOGRkNTRjYzNkN2Q1NTFjOA 密码:p5Sb

Windows版本地址：
https://space.dingtalk.com/s/gwHOABmLzwLOGlgkPQPaACBhNzNjYjI5NTYxMzQ0NmUwOWRmMTFlN2UzMTYxNDQ4Mw 密码:V7ms

## 在VirtualBox中安装Linux

https://space.dingtalk.com/s/gwHOABmLxALOGlgkPQPaACA4N2JjNmIwMGI5NWU0MGE1YjZhNTBiOGNjMDZhOTJiNQ 密码:pLgb

## 修改Linux网络配置
修改`/etc/sysconfig/network-scripts/ifcfg-eth0`，使其中的HWADDR与ip ad命令中显示的MAC地址一致，reboot后ping www.alibaba-inc.com ，检查网络是否正常，一个ifcfg-eth0文件实例如下

````
DEVICE=eth0
TYPE=Ethernet
HWADDR=08:00:27:0d:bd:b0
ONBOOT=yes
NM_CONTROLLED=yes
BOOTPROTO=dhcp
````

# 安装PouchContainer
目前PouchContainer支持两种Linux发行版：Ubuntu和CentOS，以下以CentOS 7为例，介绍PouchContainer的安装和启动。

## 安装yum-utils

使用以下命令安装所需的包yum-utils，其提供了yum-config-manager实用程序。
````
$ sudo yum install -y yum-utils
````
以下是正确的运行结果

![](https://img.alicdn.com/tfs/TB1JiVvDx9YBuNjy0FfXXXIsVXa-865-529.png)

## 配置PouchContainer库

使用以下命令添加PouchContainer库。

```shell
$ sudo yum-config-manager --add-repo http://mirrors.aliyun.com/opsx/opsx-centos7.repo
$ sudo yum update
```
以下是正确的运行结果

![](https://img.alicdn.com/tfs/TB1vmqzDv1TBuNjy0FjXXajyXXa-865-121.png)

![](https://img.alicdn.com/tfs/TB1ik01Dr9YBuNjy0FgXXcxcXXa-865-245.png)

## 安装PouchContainer

运行以下命令安装PouchContainer的最新版本。如果这是第一次在主机上安装PouchContainer，则会提示您接受GPG密钥，并显示密钥的指纹。

```
$ sudo yum install pouch
```
以下是运行结果

![](https://img.alicdn.com/tfs/TB1Up.7DXOWBuNjy0FiXXXFxVXa-865-442.png)

## 运行PouchContainer

完成上述操作后，运行PouchContainer只需运行以下命令启动pouch服务
```
$ systemctl start pouch
```

 除此以外，还需要通过pouch拉取镜像用于加载
```
$ pouch pull busybox
```

其中busybox就是在远端的一个镜像，以下是运行结果

![](https://img.alicdn.com/tfs/TB1ST38DeSSBuNjy0FlXXbBpVXa-865-532.png)

执行以下命令启动一个busybox基础容器

```
$ pouch run -t -d busybox sh
```

结果返回一串ID，用于登陆所启动的容器，执行

```
$ pouch exec -it {ID} sh
```
登入启动的容器，其中ID是上条命令输出的完整ID中的前六位。
登陆成功后，就可以在容器内正常执行shell指令了，如下所示

![](https://img.alicdn.com/tfs/TB1WX.7DXOWBuNjy0FiXXXFxVXa-865-532.png)
