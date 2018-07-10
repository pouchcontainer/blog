# PouchContainer Environment Building and Started Guide based on VirtualBox and CentOS7 for Mac

This article is mainly to guide for container developers to build pouch container environment on virtual box and centos. There are mainly devided into three parts which are be detailed next. Hopefully you will finish this task smoothly and enjoy your container travelling.

## Install Virtual Box and Create Virtual host

**In this part , it will guide you to create a virtual host step by step.**

***1.Create virtual host.***
Click "new" button to Create Virtual host and input name,typically it will automatically load the corresponding type and version.

![aeaf473194bf3e9f.png](file:///Users/wangzhen/Desktop/images/aeaf473194bf3e9f.png)

***2.Set memory size.***
The memory size can be set based on the memory size of the machine itself and the actual situation of the number of virtual hosts installed on the VirtualBox virtual machine, where I set it to 1024M.

![a4c952a5b884497a.png](file:///Users/wangzhen/Desktop/images/a4c952a5b884497a.png)

***3.Set virtual hard disk file type, distribute disk size, file postion and file size and then click next.***

![a9d55b3edc916ac8.png](file:///Users/wangzhen/Desktop/images/a9d55b3edc916ac8.png)

***4.Set virtual host.***

Select the virtual host that you want to set and click "setting","system","Main bord" in turn. In the startup sequence item, select the "floppy disk", point the right button , put "" floppy disk" "in the last of the startup sequence. Then click ok.

![d6e893987a366a3a.png](file:///Users/wangzhen/Desktop/images/d6e893987a366a3a.png)

Then, click "Memory", select "no disk" and click the optical disc icon on the right, click "select a virtual disc", and then select the image file of centos in the popup file selection window.

![db326af7e17837e5.png](file:///Users/wangzhen/Desktop/images/db326af7e17837e5.png)

![b660692ec386c082.png](file:///Users/wangzhen/Desktop/images/b660692ec386c082.png)

***5.Boot virtual host.***

Select the virtual host and click "start".

![0b36cd107adb5a30.png](file:///Users/wangzhen/Desktop/images/0b36cd107adb5a30.png)

![7b0d893bb42fa8a7.png](file:///Users/wangzhen/Desktop/images/7b0d893bb42fa8a7.png)

Set the language.

![c93d5305d2203ec4.png](file:///Users/wangzhen/Desktop/images/c93d5305d2203ec4.png)

Set Date & Time and INSTALLATION DESTINATION.

![7697c5f14891e121.png](file:///Users/wangzhen/Desktop/images/7697c5f14891e121.png)

![a3bddae5aa12791a.png](file:///Users/wangzhen/Desktop/images/a3bddae5aa12791a.png)

![b19f45f438b6318d.png](file:///Users/wangzhen/Desktop/images/b19f45f438b6318d.png)

Set Root password and click "reboot" after configuration.

![c96302fbcb5ddf4a.png](file:///Users/wangzhen/Desktop/images/c96302fbcb5ddf4a.png)

After the virtual machine reboot, login through the password you set just now.

![f097f2f9403bd0e2.png](file:///Users/wangzhen/Desktop/images/f097f2f9403bd0e2.png)

***6.Set dynamic ip address***
The above mentioned steps guide us to install CentOS in Virtual Box. However, there are nothing to show about ip information when we input **"ip addr"**.

![577e9c28b4190d90.png](file:///Users/wangzhen/Desktop/images/577e9c28b4190d90.png)

Input command:
```bash
cd /etc/sysconfig/network-scripts/
```
to enter the directory, check and confirm the network interface name of your own computer, here mine is enp0s3, everyone's computer may be different, some are eth0, some are etchs33. Then, input:
```bash
vi ifcfg-en0s3
```
to check and edit network configuration information.

![807dedc2cc2e0df5.png](file:///Users/wangzhen/Desktop/images/807dedc2cc2e0df5.png)

Here, change the ONBOOT=no to ONBOOT=yes and input **:wq** to save and exit.

![b297b4862794bacc.png](file:///Users/wangzhen/Desktop/images/b297b4862794bacc.png)

Input
```bash
service network restart
```
to restart network and input **"ip addr"** again, here we are able to see the dynamic ip address.

![e89b59c654f8b531.png](file:///Users/wangzhen/Desktop/images/e89b59c654f8b531.png)

## Install PouchContainer

**1.Install yum-utils**

To install PouchContainer, you need a maintained version of CentOS 7. WE are able to install PouchContainer through Aliyun mirrors. If you install PouchContainer for the first on a new host machine, you need to set up the PouchContainer repository. Then, you can install and update PouchContainer from repository.

![179f3ea59193aaf6.png](file:///Users/wangzhen/Desktop/images/179f3ea59193aaf6.png)

**2.Set up the PouchContainer repository**

Use the following command to add PouchContainer repository.

```bash
sudo yum-config-manager --add-repo http://mirrors.aliyun.com/opsx/opsx-centos7.repo
sudo yum update
```
![f20c286ef86aaeb8.png](file:///Users/wangzhen/Desktop/images/f20c286ef86aaeb8.png)

![6d3bd70ad2a08363.png](file:///Users/wangzhen/Desktop/images/6d3bd70ad2a08363.png)

**3. Install PouchContainer**

Run the following command to install the latest version of PouchContainer.
```bash
sudo yum install pouch
```
![c64eb6093ec833fc.png](file:///Users/wangzhen/Desktop/images/c64eb6093ec833fc.png)

##Run an pouchContainer instantiation

**1.Start the pouch container**

Run the following command to start a pouch container instantiation.
```bash
sudo systemctl start pouch
```

**2.Load a iso file to boot a pouchcontainer instantiation**

![73447fee431f185b.png](file:///Users/wangzhen/Desktop/images/73447fee431f185b.png)

**3.Start a busybox container**

Run the fllowing command to start a busybox base container.
```bash
pouch run -t -d busybox sh
```

![bb88a6c9c8746545.png](file:///Users/wangzhen/Desktop/images/bb88a6c9c8746545.png)

**4.Login container**

Input the following command:
```bash
pouch exec -it {ID} sh 
```
to login container, the ID is the first six byte of the previous output ID.

![3f5d3aa64cff67a2.png](file:///Users/wangzhen/Desktop/images/3f5d3aa64cff67a2.png)

After login, it shows like below:

![a6be140f51ca5857.png](file:///Users/wangzhen/Desktop/images/a6be140f51ca5857.png)

## Congratulation

We hope this guide would help you get up and run with PouchContainer. Enjoy it!