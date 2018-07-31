## PouchContainer Environment Setup Tutorial (PouchContainer + CentOS7 + VirtualBox) ##
### 1. download and install virtualBox ###
#### Download VirtualBox from ALILANG-Manager-Software Download, default version is 5.2.12 ####
### 2.Download the virtual machine backup CentOS of the development environment  ###
### 3.install CentOS in virtualBox###
####3.1Open VirtualBox, Click on New->Customize VM name->Choose the OS type【Linux】and version【Linus 2.6/3.x/4.x (64-bit)】-> Click on Next； ####
![avatar](https://img.alicdn.com/tfs/TB1ikQpHHGYBuNjy0FoXXciBFXa-802-461.bmp)
#### 3.2Choose 【1024M】-> Click on Next； ####
![avatar](https://img.alicdn.com/tfs/TB1RUubaO6guuRjy1XdXXaAwpXa-364-254.png)
#### 3.3Choose 【Use an existing virtual hard disk file】->Choose CentOS.vdi-> Click on Create； ####
![avatar](https://img.alicdn.com/tfs/TB1L9GaH21TBuNjy0FjXXajyXXa-364-313.png)
#### 3.4 Start VM, login using username 'root' and password 'Ali88Baiji' ####
### 4.Configuring the Internet connection ###
#### 4.1 Shut down the virtual system,Click on Setting->Select the network option->Click on the network card 1 and set the connection mode(remember the MAC address)->Click on the network card 2 and set the connection mode(remember the MAC address,it has been shown in the following ficture) ####
![avatar](https://img.alicdn.com/tfs/TB1QvicaO6guuRjy1XdXXaAwpXa-470-396.png)
![avatar](https://img.alicdn.com/tfs/TB1jFbOHMmTBuNjy1XbXXaMrVXa-357-262.png)
#### 4.2 Start VM，`ip addr`, check ip，we find that inet does not exist in enp0s3，and can not ping www.alibaba-inc.com； ####
![avatar](https://img.alicdn.com/tfs/TB1G2y0HWmWBuNjy1XaXXXCbXXa-478-260.png)
#### 4.3`cd /etc/sysconfig/network-scripts/`进入该目录 ####
![avatar](https://img.alicdn.com/tfs/TB1NOZrHHGYBuNjy0FoXXciBFXa-488-151.png)
#### 4.4 `vim ifcfg-enp0s3` ####
Add HWADDR with the mac address that you remembered, set ONBOOT=yes, and set BOOTPROTO=dhcp, save it and restart the network.

![avatar](https://img.alicdn.com/tfs/TB1xPcXHQyWBuNjy0FpXXassXXa-312-246.png)
#### 4.5 Now, the virtual machine can ping the host, but the host cannot ping the virtual machine. Now, we set the second network card. ####
under the directory of /etc/sysconfig/network-scripts/,`cp ifcfg-enp0s3 ifcfg-enp0s8`，and update it as follows：

![avatar](https://img.alicdn.com/tfs/TB1y2lCH4GYBuNjy0FnXXX5lpXa-298-265.png)
#### 4.6`service network restart` ，`ip addr`,check ip,we find it can pingwww.alibaba-inc.com； ####
![avatar](https://img.alicdn.com/tfs/TB1h6MXHQyWBuNjy0FpXXassXXa-554-188.png)
![avatar](https://img.alicdn.com/tfs/TB16XgBHKSSBuNjy0FlXXbBpVXa-554-96.png)
### 5 install PouchContainer ###
#### 5.1install the required package####
    sudo yum install -y yum-utils
#### 5.2build PouchContainer warehouse ####
    sudo yum-config-manager --add-repo http://mirrors.aliyun.com/opsx/opsx-centos7.repo
    sudo yum update
![avatar](https://img.alicdn.com/tfs/TB1TReraO6guuRjy1XdXXaAwpXa-554-97.png)
#### 5.3 install PouchContainer ####
![avatar](https://img.alicdn.com/tfs/TB1ELUuHKuSBuNjy1XcXXcYjFXa-390-93.png)
#### 5.4`systemctl start pouch` start pouch service； ####
#### 5.5`pouch run -t -d busybox sh` start the base container of busybox，it gengerates a ID； ####
![avatar](https://img.alicdn.com/tfs/TB1ykZrHHGYBuNjy0FoXXciBFXa-455-48.png)
#### 5.6`pouch exec -it {ID} sh`，ID is the first six digits in the complete ID of the previous command output； ####
![avatar](https://img.alicdn.com/tfs/TB1gaABHKSSBuNjy0FlXXbBpVXa-460-50.png)