# 基于VirtualBox + CentOS 7的PouchContainer环境搭建与上手指南

## 安装VirtualBox与CentOS 7
VirtualBox的下载地址：[https://www.virtualbox.org/wiki/Downloads](https://www.virtualbox.org/wiki/Downloads) ，根据自己的系统选择对应的版本下载，本文档使用的是macOS系统，下载完成后按照指引安装。


CentOS 7的下载地址[https://www.centos.org/download/](https://www.centos.org/download/) ，本文使用的安装包是Minimal版本。启动VirtualBox后，选择新建->自定义名称->选择类型[__Linux__]->选择版本[__Red Hat (64-bit)__]->继续->选择内存大小[__1024MB__]->继续->选择[__现在创建虚拟硬盘__]->创建->选择[__VDI (VirtualBox 磁盘映像)__]->继续->选择[__动态分配__]->继续->设置文件位置和大小->创建。

完成以上步骤后，选择刚创建的虚拟机实例，选择菜单的设置->选择存储->选择控制器: IDE这一栏的[__添加虚拟光驱__]->选择[__选择磁盘__]->选择下载好的CentOS 7的安装镜像文件->选择[__OK__]。

启动虚拟机实例，按照指引安装CentOS 7即可。

## 搭建PouchContainer运行环境
启动虚拟机后，如果无法连接网络，此时执行命令:

```
ip addr
```
查看网卡的名称，本文档虚拟机中使用的网卡是enp0s3。执行命令:
```
vi /etc/sysconfig/network-scripts/ifcfg-enp0s3
```
编辑文本，将`ONBOOT=no`修改为`ONBOOT=yes`，然后保存退出。重启网卡服务即可:
```
service network restart
```
接下来是安装和启动PouchContainer的主要过程:

1. 安装yum-utils
    ```
    sudo yum install -y yum-utils
    ```

2. 设置PouchContainer的软件源
    ```
    sudo yum-config-manager --add-repo http://mirrors.aliyun.com/opsx/opsx-centos7.repo
    sudo yum update
    ```

3. 安装PouchContainer

    ```
    sudo yum install pouch
    ```

4. 启动PouchContainer服务
    ```
    sudo systemctl start pouch
    ```

5. 启动容器
    ```
    pouch run -t -d busybox sh
    ```

6. 登入容器, 其中73cb2d是上一条命令返回ID的前六位
    ```
    pouch exec -it 73cb2d sh
    ```

7. 与容器进行交互,执行后即可看到容器输出hello world
    ```
    echo "hello world"
    ```

至此，整个流程已经完成。
