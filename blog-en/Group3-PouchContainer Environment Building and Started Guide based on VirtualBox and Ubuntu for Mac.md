---
title: PouchContainer Environment Building and Started Guide based on VirtualBox and Ubuntu for Mac
---
This article is mainly to guide for container developers to build pouch container environment on virtual box and Ubuntu. There are mainly devided into two parts which are be detailed next. Hopefully you will finish this task smoothly and enjoy your container travelling.

# PouchContainer download
In this part , it will guide you to download PouchContainer files step by step. Personal github and git are required.
git can be downloaded at:[https://www.git-scm.com/download/](https://www.git-scm.com/download/)
## 1. Fork repo from github source 
Visit [https://github.com/alibaba/pouch](https://github.com/alibaba/pouch)and sign in your personal github, click 'Fork' button at the upper right corner.
![enter description here](./images/1.2-1.png)

## 2. Download repo
### 2.1 Download by git command
Click 'Clone or download' and copy the url which will be used in the git command.
![enter description here](./images/1.2-1.png)

Open terminal in mac, choose object folder by 'cd' command and input:

``` javascript?linenums
git clone https://github.com/billhcz/pouch.git
```
## 2.2 download by ZIP files
This is a easier way to download.on your github click 'Download ZIP'.
![enter description here](./images/1.2-3.png)
Unzip the the downloaded zip file in your object folder which will be shared to the Ubuntu.

# Install Virtualbox and PouchContainer
## 1. Virtualbox configuration
Download and install VirtualBox, download Virtualbox disk file ubuntuPouch.vdi
Launch VirtualBox, click 'New' button - customize'Name' - 'Type': 'Linux' - 'Version': 'Ubuntu (64-bit)' 
![enter description here](./images/2.0-1.png)
'Continue' - 'Memory': 1024M
![enter description here](./images/2.0-2.png)
'Continue' - choose'Use an existing virtual hard disk file' - choose ubuntuPouch.vdi - 'Create'
![enter description here](./images/2.0-3.png)
Start the mechine, after signing in, switch to root user by typing:

``` javascript?linenums
sudo -i
```
## 2. Shared folder mounting
To mount the shared folder in Ubuntu, guest additions are required to installed which will take you a few minutes.
### 2.1 Guest additions installatin

Install VBoxLinuxAdditions,click menu'Devices'-'Insert Guest Additions CD image…', then input the commands by oder:

``` javascript?linenums
sudo apt-get install virtualbox-guest-dkms
sudo mount /dev/cdrom /mnt/
cd /mnt
./VBoxLinuxAdditions.run
```
![enter description here](./images/2.2.1-1.png)
![enter description here](./images/2.2.1-2.png)
![enter description here](./images/2.2.1-3.png)
![enter description here](./images/2.2.1-4.png)

When 'Do you want to continue?' occurs, just input Y

When 'VirtualBox Guest Additions: modprobe vboxsf failed' occurs, just reboot and switch to root user as the first part introduced:

``` javascript?linenums
reboot
```

### 2.2 Shared folder setting and mounting
Now, we can start the mounting. Click'Devices' - choose 'Shared Folders Settings', set your container files address as 'Folder Path', 'Folder Name' as 'share' - then tick 'Auto-mount' and 'Make Permanent' - 'OK'
![enter description here](./images/2.1-1.png)
Mount Shared folder by typing:

``` javascript?linenums
sudo mount -t vboxsf share /root/gopath/src/github.com/alibaba/
```
'share' should be the same as 'Folder Name'.

## 3. PouchContainer launching
Test network：

``` javascript?linenums
ping www.alibaba-inc.com
```
Launch pouch service (Default boot)：

``` javascript?linenums
systemctl start pouch
```
![enter description here](./images/2.3-1.png)

Launch a busybox basic container：

``` javascript?linenums
pouch run -t -d busybox sh
```
![enter description here](./images/2.3-2.png)

Log in the launched container, where ID is the first six digits of the complete ID of the previous command output:

``` javascript?linenums
pouch exec -it {ID} sh
```
![enter description here](./images/2.3-3.png)

# congratulations
We hope this guide would help you get up and run with PouchContainer. Enjoy it!