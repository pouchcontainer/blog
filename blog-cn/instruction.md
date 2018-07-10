
# 环境搭建
由于PouchContainer是企业级容器方案，故其只支持Linux操作系统。因此为了使用PouchContainer。我们采用了VirtualBox+CentOS7来进行环境的搭建。  

1、下载virtualBox和CentOS7镜像，并安装virtualBox。
  
2、启动virtualBox并新建一个虚拟机。 
 
![avator](https://img.alicdn.com/tfs/TB1V7NzDx9YBuNjy0FfXXXIsVXa-720-424.png)

3、虚拟机名称可以**自定义**，类型选择**Linux**，版本选择**Red Hat(64-bit)**。  

![avator](https://img.alicdn.com/tfs/TB1upE.DXOWBuNjy0FiXXXFxVXa-572-370.png)

4、内存大小根据配置选择**1024M**。  

![avator](https://img.alicdn.com/tfs/TB1hORzDx9YBuNjy0FfXXXIsVXa-572-370.png)

5、选择使用已有的虚拟硬盘文件，从本地导入已下载的镜像。  

![avator](https://img.alicdn.com/tfs/TB194I4DbGYBuNjy0FoXXciBFXa-572-370.png)

6、启动虚拟机，然后以管理员的身份进行登录。  

![avator](https://img.alicdn.com/tfs/TB1I_fuDDtYBeNjy1XdXXXXyVXa-720-443.png)

7、使用vim修改/etc/sysconfig/network-scripts/ifcfg-eth0，使其中的HWADDR与ip ad命令中显示的MAC地址一致，reboot后使用ping命令检查网络是否正常。  
至此，系统环境已经搭建完成，接下来进行PouchContainer的安装。  

# PouchContainer安装与启动
对于PouchContainer的安装只需要很少的步骤就能够在机器上自动安装。目前支持两种Linux发行版:Ubuntu和CentOS。因为虚拟机安装的CentOS7，因此以CentOS7来作为安装的示例。  

1、安装yum-utils  
安装所需要的包，yum-utils提供了yum-config-manager工具。  

```
sudo yum install -y yum-utils
```

2、添加PouchContainer仓库  

```
sudo yum-config-manager --add-repo http://mirrors.aliyun.com/opsx/opsx-centos7.repo
sudo yum update
```

3、安装PouchContainer  

```
sudo yum install pouch
```

4、启动PouchContainer

```
sudo systemctl start pouch
```
# 创建实例  
1、启动服务  

```
systemctl start pouch
```

2、启动一个busybox基础容器

```
pouch run -t -d busybox sh
```

3、登入启动的容器  

```
pouch exec -it {ID} sh
```
其中ID是上条命令输出的完成ID中的前六位。
