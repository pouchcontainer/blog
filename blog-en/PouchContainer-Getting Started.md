# PouchContainer - Getting Started
###Install Ubuntu16.04 (Xenial LTS)
Download Link:

https://wiki.ubuntu.com/XenialXerus/ReleaseNotes?_ga=2.66502190.1690246585.1511691893-1975959426.1511691893

To install PouchContainer, you need a maintained version of Ubuntu 16.04 (Xenial LTS). Archived versions aren't supported or tested.

PouchContainer is conflict with Docker, so you must uninstall Docker before installing PouchContainer.

**Prerequisites**

PouchContainer supports LXCFS to provide strong isolation, so you should install LXCFS firstly. By default, LXCFS is enabled.

``` bash
sudo apt-get install lxcfs
```
![Alt text](https://github.com/ProgrammingK/blog/blob/master/image/1.png)

Install packages to allow 'apt' to use a repository over HTTPS:

``` bash
sudo apt-get install curl apt-transport-https ca-certificates software-properties-common
```
![Alt text](https://github.com/ProgrammingK/blog/blob/master/image/2.png)

**1. Add PouchContainer's official GPG key**

``` bash
curl -fsSL http://mirrors.aliyun.com/opsx/pouch/linux/debian/opsx@service.alibaba.com.gpg.key | sudo apt-key add -
```
![Alt text](https://github.com/ProgrammingK/blog/blob/master/image/3.png)

**2. Set up the PouchContainer repository**

Before you install PouchContainer for the first time on a new host machine, you need to set up the PouchContainer repository. We enabled `stabel` repository by default, you always need the `stable` repository. To add the `test` repository, add the word `test` after the word `stable` in the command line below. Afterward, you can install and update PouchContainer from the repository.

``` bash
sudo add-apt-repository "deb http://mirrors.aliyun.com/opsx/pouch/linux/debian/ pouch stable"
```
![Alt text](https://github.com/ProgrammingK/blog/blob/master/image/4.png)

**3. Install PouchContainer**

Install the latest version of PouchContainer.

``` bash
# update the apt package index
sudo apt-get update
sudo apt-get install pouch
```
![Alt text](https://github.com/ProgrammingK/blog/blob/master/image/5.png)
![Alt text](https://github.com/ProgrammingK/blog/blob/master/image/6.png)

After installing PouchContainer, the `pouch` group is created, but no users are added to the group.

**4. Start PouchContainer**

``` bash
sudo service pouch start
```
![Alt text](https://github.com/ProgrammingK/blog/blob/master/image/7.png)

Afterwards, you can pull an image and run PouchContainer containers.

#### Install VirtualMachine inside VirtualBox
**1. Configure VirtualMachine**
Open VirtualBox,choose "new" - select [Linux] - version [Ubuntu (64-bit)]

![Alt text](https://github.com/ProgrammingK/blog/blob/master/image/20180723224944.png)

Continue - Memery[1024M] - Continue - choose target vdi file - Create

![Alt text](https://github.com/ProgrammingK/blog/blob/master/image/20180723225219.png)

**2. Login VirtualMachine**
Start the new instance, login with username:pouch, pwd:123456;then, switch to root user

**3. Checking Internet**
```shell
ping www.alibaba-inc.com
```
**4. Start pouch service**
Run command:
```shell
systemctl start pouch
```
pouch service default to run after the instance started
**5. Create basic container, and login**
Use:
```shell
pouch run -t -d busybox sh
```
to start a busybox basic container
The starting six chars is the ID in next command
Run:
```shell
pouch exec -it {ID} sh
```
Then you logged into the container

![Alt text](https://github.com/ProgrammingK/blog/blob/master/image/20180723225507.png)

***Caution***
You ***CAN NOT*** code in the VirtualMachine instance, if you want, please reffer to next chapter.

### Setting up Share Floder in VirtualBox
**1. Choose source floder**
choose the floder to share with the VM

![Alt text](https://github.com/ProgrammingK/blog/blob/master/image/20180723214620.png)

restart the VM instance
**2. Mount the source floder to target floder**
Run command
```shell
sudo mount -t vboxsf myPouch /root/gopath/src/github.com/alibaba/pouch
```
Any Exception like ："wrong fs type,bad option,bad superblock...“ please install the following plugins：
```shell
sudo apt install nfs-common
sudo apt install cifs-utils
sudo apt install virtualbox-guest-utils
```

after below commands,still getting exception like: “no such devive......” Run command ：
```shell
modprobe -a vboxguest vboxsf vboxvideo
```
See also: https://wiki.archlinux.org/index.php/VirtualBox

**3. Test**
Insert a useless file at source floder, if you can find the file in target floder, successed!