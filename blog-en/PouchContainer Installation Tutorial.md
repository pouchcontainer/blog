
# PouchContainer installation tutorial

## Environment and preparation


- Host system: CentOS 7 / Ubuntu 16.04
- Required software: VirtualBox
- How to get the Virtualbox package:
Use the Arirang-Housekeeper-Office software to manage the installation of VirtualBox. The default version is 5.2.22. If you don't have Arirang, please download it on the DingTalk plate.

Mac version address:
[Mac-address](https://space.dingtalk.com/s/gwHOABma4QLOGlgkPQPaACBiMzk5ZWRjZTAyOGI0MTBkOGRkNTRjYzNkN2Q1NTFjOA)
&emsp;&emsp;&emsp;passwd: p5Sb

Windows version address:
[Windows-address](https://space.dingtalk.com/s/gwHOABmLzwLOGlgkPQPaACBhNzNjYjI5NTYxMzQ0NmUwOWRmMTFlN2UzMTYxNDQ4Mw) 
&emsp;&emsp;&emsp;passwd: V7ms


## Loading virtual machines and configurations

### Virtual machine loading
1. Download the image in the DingTalk group, either CentOS or Ubuntu
2. Open Virtualbox and click `New`
3. Set the name of the virtual machine by yourself, the type is `Linux`, and the version is `Linux 2.6/3.x/4.x (64-bit)`.
4. Recommended memory size, the default `1024`
5. Select `Use existing virtual hard disk file`, select the downloaded virtual machine image, and click `Create`.
6. Start virtual machine

### Virtual Machine Configuration

#### CentOS
username：root

passwd：Ali88Baiji

Try to ping www.alibaba-inc.com.

If success, go to the next step.

If not, modify the NIC information and use the `ip ad` command to view the mac address assigned to the virtual machine NIC by Virtualbox, as shown in the following figure:

![avatar](https://images2018.cnblogs.com/blog/761101/201807/761101-20180731012815511-1807884097.png)


Use the following command to modify the IP address of the current virtual machine:

```
vi /etc/sysconfig/network-scripts/ifcfg-eth0
```
Modify the value corresponding to `HWADDR=` and change it to your own query result (the result in the red box).
#### Ubuntu
username：pouch

passwd：123456

Try to ping www.alibaba-inc.com.

If  success, go to the next step.



## Install PouchContainer
### CentOS
#### 1. Install the required package yum-utils

```
sudo yum install -y yum-utils
``` 
result：
![avatar](https://images2018.cnblogs.com/blog/761101/201807/761101-20180731011533827-586132642.png)


#### 2. add PouchContainer repo

```
sudo yum-config-manager --add-repo http://mirrors.aliyun.com/opsx/opsx-centos7.repo
sudo yum update
``` 
result：

![avatar](https://images2018.cnblogs.com/blog/761101/201807/761101-20180731011449985-103277584.png)


#### 3. install PouchContainer

```
sudo yum install pouch
``` 
result：

![avatar](https://images2018.cnblogs.com/blog/761101/201807/761101-20180731011458334-832954266.png)


#### 4. start PouchContainer

```
//Set boot
systemctl start pouch
//Pull remote mirror
pouch pull busybox
``` 
result：

![avatar](https://images2018.cnblogs.com/blog/761101/201807/761101-20180731011509633-1996626722.png)


```
//Start the image, the first 6 characters are the ID required for login
pouch run -t -d busybox sh
```
result：

![avatar](https://images2018.cnblogs.com/blog/761101/201807/761101-20180731011519379-266427941.png)



```
//Log in to the image and execute some commands
pouch exec -it {ID} sh
```

result：

![avatar](https://images2018.cnblogs.com/blog/761101/201807/761101-20180731011527313-674343869.png)


### Ubuntu

#### 1.install LXCFS

```
sudo apt-get install lxcfs
```

result：

![avatar](https://images2018.cnblogs.com/blog/761101/201807/761101-20180731011541046-1004355148.png)

#### 2.Install packages to allow 'apt' to use a repository over HTTPS:

```
sudo apt-get install curl apt-transport-https ca-certificates software-properties-common
```

result：

![avatar](https://images2018.cnblogs.com/blog/761101/201807/761101-20180731011649030-1905207872.png)

#### 3.add PouchContainer's official GPG key

```
curl -fsSL http://mirrors.aliyun.com/opsx/pouch/linux/debian/opsx@service.alibaba.com.gpg.key | sudo apt-key add -
```

result：

![avatar](https://images2018.cnblogs.com/blog/761101/201807/761101-20180731011700640-614313500.jpg)


#### 4.Verify that you now have the key with the fingerprint F443 EDD0 4A58 7E8B F645 9C40 CF68 F84A BE2F 475F, by searching for the last 8 characters of the fingerprint.

```
apt-key fingerprint BE2F475F
```

result：

![avatar](https://images2018.cnblogs.com/blog/761101/201807/761101-20180731011710188-616788657.jpg)

#### 5.add PouchContainer repo

```
sudo add-apt-repository "deb http://mirrors.aliyun.com/opsx/pouch/linux/debian/ pouch stable"
```

result：

![avatar](https://images2018.cnblogs.com/blog/761101/201807/761101-20180731011718977-2015734432.jpg)


#### 6.install PouchContainer

```
sudo apt-get update
sudo apt-get install pouch
```

result：

![avatar](https://images2018.cnblogs.com/blog/761101/201807/761101-20180731011828447-1398873692.jpg)

#### 7.start PouchContainer(such as CentOS)

## shh remote login & file sharing

### shh remote login

#### 1.Configure the NIC to be in bridge mode
The virtualbox NAT only supports one-way communication from the virtual machine to the host. That is, the virtual machine can ping the host, but the host cannot ping the virtual machine, so the network configuration needs to be changed to the bridge mode.

![avatar](https://cdn.nlark.com/lark/0/2018/png/123186/1532952006217-7fd4751c-f246-4008-8241-ae85af32ecf0.png)

#### 2.Install the ssh service and start it. The default port is 22 

```
apt-get install openssh-server
```
![avatar](https://images2018.cnblogs.com/blog/761101/201807/761101-20180731094405012-1522115623.png)

#### 3.Check the ssh service is started.

![avatar](https://cdn.nlark.com/lark/0/2018/png/123186/1532952211218-fa86e7f1-3701-4cb9-bebd-c19bffed324c.png)

#### 4.Get virtual machine IP

![avatar](https://cdn.nlark.com/lark/0/2018/png/123186/1532952133466-32a66208-df68-47c7-9124-6d9dc8597a7f.png)

#### 5.ssh remote login  
 
```
ssh usrname@IP
```
![avatar](https://cdn.nlark.com/lark/0/2018/png/123186/1532952276881-e3af58c4-7f23-4f40-81e1-b9a187ec9f5d.png)




### File Sharing

#### 1.Configure shared folder virtualbox -> settings -> shared folder

![avatar](https://cdn.nlark.com/lark/0/2018/png/123186/1532952361541-99ba9094-7f9c-4a3b-ba7e-9822a2196ad7.png)

#### 2.Install Guest Additions
If you want to use the shared folder feature that comes with VirtualBox, you must first install Guest Additions.

Installation method: Top menu bar -> devices-> Install Guest Additions

![avatar](https://cdn.nlark.com/lark/0/2018/png/123186/1532952462469-cbd198d6-ac62-4565-ae4c-e76e0ebd8355.png)

set the first IDE slave channel

![avatar](https://cdn.nlark.com/lark/0/2018/png/123186/1532952473708-06dcb074-1358-48d5-bcb1-ddd590461790.png)

#### 3.File mount
If you encounter a problem with 'unknown filesystem type "vboxsf"', execute the following command:

```
apt-get update
apt-get install virtualbox-guest-utils
```

![avatar](https://cdn.nlark.com/lark/0/2018/png/123186/1532952524332-240c1604-9e57-464e-8132-fcecdc872906.png)

Then restart the virtual machine and remount file.

![avatar](https://cdn.nlark.com/lark/0/2018/png/123186/1532951986751-fb248032-4172-4a34-87ac-270fef0f75da.png)

