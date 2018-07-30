# Introduction
PouchContainer is a high-efficiency and enterprise-class container engine with the characteristics of strong isolation, high portability and low resource consumption. Currently, PouchContainer only supports the Linux operating system, so a virtual machine is required to run and test locally. This guide instructs the readers to build a PouchContainer environment  based on VirtualBox + Ubuntu 16.04 on Mac or Windows.


# Installing VirtualBox
__1. You can choose one of the following ways to install Virtualbox:__
* Search virtualbox from AliLlang (Manager->Software download) , download it, then install
* Download  virtualbox from [VirtualBox](https://download.virtualbox.org/virtualbox/5.2.16/VirtualBox-5.2.16-123759-OSX.dmg),  then install
<strong>2. After finished, double-click VirtualBox.pkg and click </strong><strong><em>Continue </em></strong><strong>to install. </strong>



![image.png | left | 564x376](https://cdn.nlark.com/lark/0/2018/png/135654/1532965787724-f8873d75-0db0-44ae-9e76-38e463193304.png "")

__3.Customizable installation is available__


![image.png | left | 556x370](https://cdn.nlark.com/lark/0/2018/png/135654/1532965968099-0b8e72cc-fe70-452a-a5db-2a8c4fd63d63.png "")


# Configuring VirtualBox
<strong>1、Open </strong><span data-type="color" style="color:rgb(38, 38, 38)"><span data-type="background" style="background-color:rgb(255, 255, 255)"><strong>VirtualBox，</strong></span></span><span data-type="color" style="color:rgb(36, 41, 46)"><span data-type="background" style="background-color:rgb(255, 255, 255)"><strong>Click </strong></span></span><span data-type="color" style="color:rgb(36, 41, 46)"><span data-type="background" style="background-color:rgb(255, 255, 255)"><em><strong>New</strong></em></span></span><span data-type="color" style="color:rgb(36, 41, 46)"><span data-type="background" style="background-color:rgb(255, 255, 255)"><strong> in the VirtualBox Manager to create a new virtual machine</strong></span></span>


![image.png | left | 265x68](https://cdn.nlark.com/lark/0/2018/png/135654/1532966022381-d6c1f736-be1e-4a84-93ca-f14f33a85d75.png "")

__2、Input the name, systemm type and version of the new virtual machine.__
* <span data-type="color" style="color:rgb(36, 41, 46)"><span data-type="background" style="background-color:rgb(255, 255, 255)"><em>Choosing a desired name</em></span></span>
* <span data-type="color" style="color:rgb(36, 41, 46)"><span data-type="background" style="background-color:rgb(255, 255, 255)"><em>Linux</em></span></span><span data-type="color" style="color:rgb(36, 41, 46)"><span data-type="background" style="background-color:rgb(255, 255, 255)"> type</span></span>
* <span data-type="color" style="color:rgb(36, 41, 46)"><span data-type="background" style="background-color:rgb(255, 255, 255)"><em>Ubuntu (64-bit) </em></span></span><span data-type="color" style="color:rgb(36, 41, 46)"><span data-type="background" style="background-color:rgb(255, 255, 255)">version</span></span>


![image.png | left | 538x485](https://cdn.nlark.com/lark/0/2018/png/135654/1532966206015-a825b620-aff5-49b2-b630-9a7302333a8a.png "")

<strong>3、Set the memory size of the virtual machine, or choosing the default size(1024 MB). </strong><span data-type="color" style="color:rgb(36, 41, 46)"><span data-type="background" style="background-color:rgb(255, 255, 255)"><strong> Then click </strong></span></span><span data-type="color" style="color:rgb(36, 41, 46)"><span data-type="background" style="background-color:rgb(255, 255, 255)"><strong><em>Next</em></strong></span></span><span data-type="color" style="color:rgb(36, 41, 46)"><span data-type="background" style="background-color:rgb(255, 255, 255)"><strong>.</strong></span></span>


![image.png | left | 551x485](https://cdn.nlark.com/lark/0/2018/png/135654/1532966237848-4134d823-745d-46ca-bc7b-bb91a029c478.png "")

<strong>4、</strong><span data-type="color" style="color:rgb(25, 31, 37)"><span data-type="background" style="background-color:rgb(255, 255, 255)"><strong>Virtual hard disk settings</strong></span></span>
* Choose <span data-type="color" style="color:rgb(25, 31, 37)"><span data-type="background" style="background-color:rgb(255, 255, 255)"><em>Using the existing virtual hard disk file</em></span></span><span data-type="color" style="color:rgb(25, 31, 37)"><span data-type="background" style="background-color:rgb(255, 255, 255)">,  click to choose the file (choose ubuntuPouch.vdi, this Ubuntu has been installed PouchContainer).</span></span>
* <span data-type="color" style="color:rgb(25, 31, 37)"><span data-type="background" style="background-color:rgb(255, 255, 255)"> If there is no such a file, click </span></span><span data-type="color" style="color:rgb(25, 31, 37)"><span data-type="background" style="background-color:rgb(255, 255, 255)"><em>create the virtual hard disk now</em></span></span><span data-type="color" style="color:rgb(25, 31, 37)"><span data-type="background" style="background-color:rgb(255, 255, 255)">, and you can configure the system itself and manually install the PouchContainer. Then click </span></span><span data-type="color" style="color:rgb(25, 31, 37)"><span data-type="background" style="background-color:rgb(255, 255, 255)"><em>create</em></span></span><span data-type="color" style="color:rgb(25, 31, 37)"><span data-type="background" style="background-color:rgb(255, 255, 255)">.</span></span>


![image.png | left | 564x492](https://cdn.nlark.com/lark/0/2018/png/135654/1532966506742-18f40273-a294-47cd-b799-bcaf115579d3.png "")

# Enter the virtual machine, start and log in the pouch.
<strong>1、Click </strong><strong><em>start</em></strong><strong>, input the original username and password </strong>
```
username：pouch
password：123456
```
__2、Switch to the root user__
```
sudo -i
```
__3、Check network connection__
```
ping www.alibaba-inc.com
```
__4、Start the pouch service (default boot) __
```
systemctl start pouch  //If not succeed, check the network again   
```
__5、Start an existing busybox base container__
```
pouch run -t -d busybox sh
```
__6、Do pull if no busybox available__
```
pouch pull busybox
```
__7、Log in the started container, the ID is the first 6 digits of the complete IDs from the previous __
```
pouch exec -it {ID} sh
```
command output


![image.png | left | 747x113](https://cdn.nlark.com/lark/0/2018/png/135654/1532967272649-b1548c1e-bd10-4487-8dc1-64773bf929fe.png "")

# Connecting Mac terminal with Virtualbox
Using a virtual machine created by Virtualbox while working can effectively reduce potential interference between the local environment and the working environment.  However, the UI of Server is not friendly. Using SSH to connect to virtual machine and local terminals to edit is a suggesting alternative.
__1、Port forwarding__
1) Click *Setting*, *Internet, Advanced and Port forwarding*


![image.png | left | 568x395](https://cdn.nlark.com/lark/0/2018/png/135654/1532967358783-7cb8dc18-6b20-47bd-9fdb-1a2f60d531dc.png "")

2)Click the icon in red, and set host post *9022， *subsystem port *22*. Then save.


![image.png | left | 544x393](https://cdn.nlark.com/lark/0/2018/png/135654/1532967432115-464b2478-dd49-4aa8-ba66-21cca6c1c8fd.png "")

3)Local terminal connection

A successful connection interface will be


![image.png | left | 614x289](https://cdn.nlark.com/lark/0/2018/png/135654/1532967488069-5a46f6aa-b9c0-4a98-9cb8-feaceaa4c53c.png "")

Until now, you have finished the environment construction and you can operate on the terminal. Please enjoy your journey on PouchContainer!

