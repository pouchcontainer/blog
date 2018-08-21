# PouchContainer Environment Building and Started Guide based on VirtualBox and CentOS7 for Mac

This article is mainly to guide for container developers to build pouch container environment on virtual box and centos. There are mainly devided into three parts which are be detailed next. Hopefully you will finish this task smoothly and enjoy your container travelling.

## Install Virtual Box and Create Virtual host

**In this part , it will guide you to create a virtual host step by step.**

***1.Create virtual host.***
Click "new" button to Create Virtual host and input name,typically it will automatically load the corresponding type and version.

<img width="630" height="430" src="https://img.alicdn.com/tfs/TB13Q5qDACWBuNjy0FaXXXUlXXa-1218-902.jpg"/>

***2.Set memory size.***
The memory size can be set based on the memory size of the machine itself and the actual situation of the number of virtual hosts installed on the VirtualBox virtual machine, where I set it to 1024M.

<img width="630" height="430" src="https://img.alicdn.com/tfs/TB1WOXlDpGWBuNjy0FbXXb4sXXa-1278-882.jpg"/>

***3.Set virtual hard disk file type, distribute disk size, file postion and file size and then click next.***

<img width="630" height="430" src="https://img.alicdn.com/tfs/TB1.KieDxGYBuNjy0FnXXX5lpXa-1492-856.png"/>

***4.Set virtual host.***

Select the virtual host that you want to set and click "setting","system","Main bord" in turn. In the startup sequence item, select the "floppy disk", point the right button , put "" floppy disk" "in the last of the startup sequence. Then click ok.

<img width="630" height="430" src="https://img.alicdn.com/tfs/TB15TdDDv9TBuNjy0FcXXbeiFXa-1740-996.png"/>

Then, click "Memory", select "no disk" and click the optical disc icon on the right, click "select a virtual disc", and then select the image file of centos in the popup file selection window.

<img width="630" height="430" src="https://img.alicdn.com/tfs/TB1WYR6DuSSBuNjy0FlXXbBpVXa-1732-800.png"/>

<img width="630" height="430" src="https://img.alicdn.com/tfs/TB1tR9LDrSYBuNjSspfXXcZCpXa-1318-728.png"/>

***5.Boot virtual host.***

Select the virtual host and click "start".

<img width="630" height="430" src="https://img.alicdn.com/tfs/TB1XumeDxGYBuNjy0FnXXX5lpXa-1388-1244.png"/>

Set the language.

<img width="630" height="430" src="https://img.alicdn.com/tfs/TB1xKxcDrGYBuNjy0FoXXciBFXa-2042-1608.png"/>

Set Date & Time and INSTALLATION DESTINATION.

<img width="630" height="430" src="https://img.alicdn.com/tfs/TB1mvdtvgKTBuNkSne1XXaJoXXa-2038-1420.png"/>

<img width="630" height="430" src="https://img.alicdn.com/tfs/TB1V_4lDwmTBuNjy1XbXXaMrVXa-2038-1308.jpg"/>


Set Root password and click "reboot" after configuration.

<img width="630" height="430" src="https://img.alicdn.com/tfs/TB17axjDpOWBuNjy0FiXXXFxVXa-2038-1568.png"/>

After the virtual machine reboot, login through the password you set just now.

<img width="630" height="430" src="https://img.alicdn.com/tfs/TB1yutcDrGYBuNjy0FoXXciBFXa-1440-886.png"/>

***6.Set dynamic ip address***
The above mentioned steps guide us to install CentOS in Virtual Box. However, there are nothing to show about ip information when we input **"ip addr"**.

<img width="630" height="430" src="https://img.alicdn.com/tfs/TB148NkDuSSBuNjy0FlXXbBpVXa-1440-886.png"/>

Input command:
```bash
cd /etc/sysconfig/network-scripts/
```
to enter the directory, check and confirm the network interface name of your own computer, here mine is enp0s3, everyone's computer may be different, some are eth0, some are etchs33. Then, input:
```bash
vi ifcfg-en0s3
```
to check and edit network configuration information.

<img width="630" height="430" src="https://img.alicdn.com/tfs/TB1F1ieDxGYBuNjy0FnXXX5lpXa-1440-886.png"/>

Here, change the ONBOOT=no to ONBOOT=yes and input `:wq` to save and exit.

<img width="630" height="430" src="https://img.alicdn.com/tfs/TB1V7aODv1TBuNjy0FjXXajyXXa-1440-886.png"/>

Input
```bash
service network restart
```
to restart network and input `ip addr` again, here we are able to see the dynamic ip address.

<img width="630" height="430" src="https://img.alicdn.com/tfs/TB1eDhDDv9TBuNjy0FcXXbeiFXa-2068-1274.png"/>

## Install PouchContainer

**1.Install yum-utils**

To install PouchContainer, you need a maintained version of CentOS 7. WE are able to install PouchContainer through Aliyun mirrors. If you install PouchContainer for the first on a new host machine, you need to set up the PouchContainer repository. Then, you can install and update PouchContainer from repository.
``` bash
sudo yum install -y yum-utils
```

<img width="630" height="430" src="https://img.alicdn.com/tfs/TB1mpY2DqmWBuNjy1XaXXXCbXXa-1440-886.png"/>

**2.Set up the PouchContainer repository**

Use the following command to add PouchContainer repository.

```bash
sudo yum-config-manager --add-repo http://mirrors.aliyun.com/opsx/opsx-centos7.repo
sudo yum update
```
<img width="630" height="430" src="https://img.alicdn.com/tfs/TB1c5NPDuOSBuNjy0FdXXbDnVXa-1440-886.png"/>

<img width="630" height="430" src="https://img.alicdn.com/tfs/TB1nptADv9TBuNjy0FcXXbeiFXa-1440-886.png"/>

**3. Install PouchContainer**

Run the following command to install the latest version of PouchContainer.
```bash
sudo yum install pouch
```
<img width="630" height="430" src="https://img.alicdn.com/tfs/TB1nptADv9TBuNjy0FcXXbeiFXa-1440-886.png"/>

##Run an pouchContainer instantiation

**1.Start the pouch container**

Run the following command to start a pouch container instantiation.
```bash
sudo systemctl start pouch
```

**2.Load a iso file to boot a pouchcontainer instantiation**
```bash
pouch pull busybox
```

<img width="630" height="430" src="https://img.alicdn.com/tfs/TB1LuHGDDtYBeNjy1XdXXXXyVXa-1440-886.png"/>

**3.Start a busybox container**

Run the fllowing command to start a busybox base container.
```bash
pouch run -t -d busybox sh
```

<img width="630" height="430" src="https://img.alicdn.com/tfs/TB1wuHGDDtYBeNjy1XdXXXXyVXa-1440-886.png"/>

**4.Login container**

Input the following command:
```bash
pouch exec -it {ID} sh 
```
to login container, the ID is the first six byte of the previous output ID.

<img width="630" height="430" src="https://img.alicdn.com/tfs/TB1miBlDuuSBuNjy1XcXXcYjFXa-1440-886.png"/>

After login, it shows like below:

<img width="630" height="430" src="https://img.alicdn.com/tfs/TB1WyxlDuuSBuNjy1XcXXcYjFXa-1440-886.png"/>

## Congratulation

We hope this guide would help you get up and run with PouchContainer. Enjoy it!