# Manual Guidance for VirtualBox + CentOS7 based PouchContainer 

### What is PouchContainer?
PouchContainer is an open-source project created by Alibaba Group. It is lightweight at enterprise's large scale with strong isolation, high portability, and minimal overhead.

### Environment Selection
Because *PouchContainer* is an enterprise-level container solution which only supports the Linux operating system, a virtual machine is needed to run and test locally. This article chooses *VirtualBox* as the virtualization platform. *CentOS (Community Enterprise Operating System)* is one of the *Linux*-based distributions for community enterprise operating systems. This article uses *CentOS7* to install an operating system for virtual machines.

## Steps for Environment Building
### VirtualBox Download
* Use Alilang - manager - software management - installation * VirtualBox *, the default version is 5.2.12.
* If you don't have Alilang, you can download it on the Ding storage.

Download link for *Mac* version:
[Https://space.dingtalk.com/gwHOABma4QLOGlgkPQPaACBiMzk5ZWRjZTAyOGI0MTBkOGRkNTRjYzNkN2Q1NTFjOA](https://space.dingtalk.com/gwHOABma4QLOGlgkPQPaACBiMzk5ZWRjZTAyOGI0MTBkOGRkNTRjYzNkN2Q1NTFjOA) Password: p5Sb

Download link for *Windows* version:
[Https://space.dingtalk.com/s/gwHOABmLzwLOGlgkPQPaACBhNzNjYjI5NTYxMzQ0NmUwOWRmMTFlN2UzMTYxNDQ4Mw](https://space.dingtalk.com/s/gwHOABmLzwLOGlgkPQPaACBhNzNjYjI5NTYxMzQ0NmUwOWRmMTFlN2UzMTYxNDQ4Mw) Password: V7ms

###CentOS Download
Download link for *CentOS7-Minimal* mirror installation:
[Https://space.dingtalk.com/s/gwHOABmslALOGlgkPQPaACAwNTg4YTBjOGI4OTI0MGQ5YjE5MDgyYWFjMzAxMDY1MQ](https://space.dingtalk.com/s/gwHOABmslALOGlgkPQPaACAwNTg4YTBjOGI4OTI0MGQ5YjE5MDgyYWFjMzAxMDY1MQ) Password: tkD3

## Environmental installation steps
###1.*Windows* installation steps
The first step, the installation package is downloaded, start the installation, click "Next".

![](https://i.imgur.com/JgZy07D.jpg)

The second step, custom installation, choose the way to install the function, the default one is ok. Select the installation location, click "Next".

![](https://i.imgur.com/j2Xa2zs.jpg)

The third step, custom installation, the default is ok. Click "Next".

![](https://i.imgur.com/EVhZFMv.jpg)

The fourth step, the web interface, click "Yes".

![](https://i.imgur.com/zMWg5Uc.jpg)

The fifth step, the installation begins, click "Install".

![](https://i.imgur.com/73TH8ZA.jpg)

The sixth step, click "Finish", the installation is completed!

![](https://i.imgur.com/PN1jAlJ.jpg)

###2.*Mac* Installation Steps

#Virtual machine new instance steps

The first step, open the "* Oracle VM VirtualBox *", click the button "New" on the upper left corner to create a new project.

![](https://i.imgur.com/axy1O43.jpg)

The second step, deploy a new virtual computer, customize the name, type "*Linux*", version: "*Red Hat*(64-bit)", memory size 1024M, select "Create virtual hard disk now", click "Create" .

**note! ! There are some windows systems that do not have a 64bit option on the version selection. The reason is that the BIOS setting of the msconfig does not turn on virtualization. Solution: Restart - press F2 immediately after starting (different computers may be different), enter BIOS - find "Security" option - choose "Virtualization" - change the setting option to "Enabled" - after restarting again, "Oracle VM VirtualBox" can run successfully**

![](https://i.imgur.com/pKuRT2S.jpg)

The third step, create a virtual hard disk, select the file location, the default size. For "virtual hardware file type", select "VDI". For "storage on the physical hard disk", select "dynamic allocation", click "create", then the project is created.

![](https://i.imgur.com/j3S5Z5T.jpg)

The fourth step, select the created project and click "Settings" to deploy the downloaded *CentOS7* mirror file.

![](https://i.imgur.com/mvEfC09.jpg)

The fifth step, select "storage", click on "controller" in the "storage medium": IDE "no disc", its attribute is assigned to the optical drive, click the disc icon on the right, add the downloaded path of *CentOS7*, click "ok" to finish setting.

![](https://i.imgur.com/92Pgf24.jpg)

**P.S:
If you have downloaded the virtual hard disk file from the link
[Https://space.dingtalk.com/s/gwHOABmLxALOGlgkPQPaACA4N2JjNmIwMGI5NWU0MGE1YjZhNTBiOGNjMDZhOTJiNQ](https://space.dingtalk.com/s/gwHOABmLxALOGlgkPQPaACA4N2JjNmIwMGI5NWU0MGE1YjZhNTBiOGNjMDZhOTJiNQ) Password: pLgb
, just select "Use existing virtual hard disk file" in the second step directly, then skip the third, fourth, and fifth steps and then finish **

## Virtual machine startup
After completing the basic settings of the virtual machine, you can start the virtual machine, select the created project, click "Start", open the virtual machine, click the black area, use the arrow up or down to select the first one.

![](https://i.imgur.com/2WkLfea.jpg)

Wait for the file to load. After the loading is completed, the following interface is displayed. You can select the language and click "OK".

![](https://i.imgur.com/1HKTQUX.jpg)

A summary of the installation information appears, wait for the icon to change from gray to black, and after "Start Installation" is lit, click "Start Installation"
   
![](https://i.imgur.com/u4MEYOE.jpg)

Click "*ROOT*Password" in the user setting, set the username to be *root*, set the password to be *Ali88Baiji*, then you can complete the "User Creation", and finally click "Finish"

![](https://i.imgur.com/x7hnRdf.jpg)

Once the installation is completed, click "Restart" to get started.

## *PouchContainer* installation
To install *PouchContainer* on a new virtual machine, first you need to install the yum-utils package for deployment, but since *CentOS* is not connected to the Internet by default, you must first connect the virtual machine as follows:
Start the virtual machine, enter the directory * cd / etc / sysconfig / network-scripts / *

![](https://i.imgur.com/fF4QnmB.jpg)

Check the network card deployment file, * ONBOOT = no * read * ONBOOT = yes *

![](https://i.imgur.com/gd9OH5f.jpg)

At this point, the virtual machine has successfully connected to the Internet, and can be verified by ping.

At the same time, in order the go through the later operations more smoothly, we can pre-install some necessary packages:  
*yum install automake autoconf git pkg-config make gcc golang qemu aclocal libseccomp-devel -y*  
Then we can install the *Pouch* package:  
*sudo yum install -y yum-utils*  
*sudo yum-config-manager --add-repo http://mirrors.aliyun.com/opsx/opsx-centos7.repo*  
* Sudo yum update *  
*sudo yum install pouch*  
At the same time, you can also do the following:  
*mkdir -p $GOPATH/src/github.com/alibaba/*  
*cd $GOPATH/src/github.com/alibaba/*  
*git clone https://github.com/alibaba/pouch.git*  
Finally, the installation of *PouchContainer* is completed!

#*PouchContainer* startup
At this point, we can achieve our ultimate goal: run *PouchContainer* by typing:  
*sudo systemctl start pouch*  
To create an interactive container project, enter the following command into the terminal:
First pull an image, taking busybox as an example:
*pouch pull busybox*
Then you can use  *pouch run -i busybox* to create an interactive container project!
