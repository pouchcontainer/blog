## 1. PouchContainer Introduction
1. PouchContainer is Alibaba Group's open source, efficient and lightweight enterprise class rich container engine technology, which can help enterprises quickly improve server utilization efficiency. PouchContainer has been used in Alibaba for many years and has strong reliability and stability.

## 2. Install VirtualBox
1. VirtualBox can be downloaded at  https://download.virtualbox.org/virtualbox/5.2.16/VirtualBox-5.2.16-123759-OSX.dmg . After the download is complete, open the VirtualBox-5.2.16-123759-OSX.dmg file and double-click VirtualBox.pkg to install it. If you need a custom installation, you can choose a custom installation.
![v1.png](https://luolivirtualbox1.oss-cn-beijing.aliyuncs.com/v1.png)

## 3. Install Ubuntu16.04 in VirtualBox
1. Open VirtualBox, first click the New button in the title bar, create a new operating system, name can be customized, type select Linux, version select Ubuntu (64-bit).
![v2.png](https://luolivirtualbox1.oss-cn-beijing.aliyuncs.com/v2.png)
2. Click the Continue button to enter the memory selection page. The default memory is 1024MB, of course, you can also increase the memory as needed.
![v3.png](https://luolivirtualbox1.oss-cn-beijing.aliyuncs.com/v3.png)
3. Click the Continue button to enter the hard disk selection page and select "Create a virtual hard disk now".
![v4.png](https://luolivirtualbox1.oss-cn-beijing.aliyuncs.com/v4.png)
4. Click the Create button and select VDI (VirtualBox Disk Image).
![v5.png](https://luolivirtualbox1.oss-cn-beijing.aliyuncs.com/v5.png)
5. Click the Continue button and select Dynamically allocated so that the virtual machine can dynamically allocate space.
![v6.png](https://luolivirtualbox1.oss-cn-beijing.aliyuncs.com/v6.png)
6. Click the Continue button to save the file in the directory of your choice. Click the Create button and a virtual machine will succeed.
![v7.png](https://luolivirtualbox1.oss-cn-beijing.aliyuncs.com/v7.png)
7. Go to the created virtual machine settings page and load the ISO file you downloaded from Storage.
![v8.png](https://luolivirtualbox1.oss-cn-beijing.aliyuncs.com/v8.png)
8. Click to start the virtual machine. Ubuntu needs to configure the system according to its own preferences (setting the language, username, password, etc.).

## 4. Install PouchContainer
1. Open the already installed ubuntu in the virtualBox and enter the username and password to log in.
2. PouchContainer relies on LXCFS to provide strong dependency guarantees, so you first need to install LXCFS, the command is as follows:
```
sudo apt-get install lxcfs
```
![p1.png](https://luolivirtualbox1.oss-cn-beijing.aliyuncs.com/p1.png)
3. Install packages to allow 'apt' to use a repository over HTTPS, the command is as follows:
```
sudo apt-get install curl apt-transport-https ca-certificates software-properties-common
```
![p2.png](https://luolivirtualbox1.oss-cn-beijing.aliyuncs.com/p2.png)
4. Add the official GPG key for PouchContainer. The command is as follows:
```
curl -fsSL http://mirrors.aliyun.com/opsx/pouch/linux/debian/opsx@service.alibaba.com.gpg.key | sudo apt-key add -
```
5. Verify that you have the F443 EDD0 4A58 7E8B F645 9C40 CF68 F84A BE2F 475F key by searching for the last 8 bits of the key BE2F475F. The command is as follows:
```
apt-key fingerprint BE2F475F
```
![p3.png](https://luolivirtualbox1.oss-cn-beijing.aliyuncs.com/p3.png)
6. When installing PouchContainer on a new host, you need to set the default mirror repository. PouchContainer allows the stable repository  to be set as the default repository. The command is as follows:
```
sudo add-apt-repository "deb http://mirrors.aliyun.com/opsx/pouch/linux/debian/ pouch stable"
```
![p4.png](https://luolivirtualbox1.oss-cn-beijing.aliyuncs.com/p4.png)
7. Download the latest PouchContainer via apt-get, the command is as follows:
```
sudo apt-get update
sudo apt-get install pouch
```
![p5.png](https://luolivirtualbox1.oss-cn-beijing.aliyuncs.com/p5.png)
8. Start PouchContainer with the following command:
```
sudo service pouch start
```
9. Download the busybox image file with the following command:
```
pouch pull busybox
```
10. Run busybox, the command is as follows:
```
pouch run -t -d busybox sh
```
11. After the container runs successfully, it will output the ID of the container. According to this ID, enter the container of the busybox. The command is as follows:
```
pouch exec -it 23f06f sh
```
![p7.png](https://luolivirtualbox1.oss-cn-beijing.aliyuncs.com/p7.png)
12. This allows you to interact with the container inside the container. Enter Exit to exit the container after the interaction is complete.

## 5. Precautions
1. PouchContainer conflicts with Docker. Before installing PouchContainer, you need to check if there is Docker, otherwise the installation will fail.

## 6. Summary
1. Through the above tutorial, we can easily experience PouchContainer on non-Linux computers.