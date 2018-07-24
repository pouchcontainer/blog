# Configuring Experience Environment of PouchContainer Based on VirtualBox and Ubuntu16.04 
## Download and installation preparation
### Download the virtual machine backup of the development environment
- Please download the [ubuntuPouch.vdi](https://www.virtualbox.orgubuntuPouch.vdi). It is recommended to download this file in advance because it is relatively large (3.2G). 
### Download and install VirtualBox

- The links are as follows:
  + [Link of Mac version](https://space.dingtalk.com/s/gwHOABma4QLOGlgkPQPaACBiMzk5ZWRjZTAyOGI0MTBkOGRkNTRjYzNkN2Q1NTFjOA) password：p5Sb
  + [Link of Windows version](https://space.dingtalk.com/s/gwHOABmLzwLOGlgkPQPaACBhNzNjYjI5NTYxMzQ0NmUwOWRmMTFlN2UzMTYxNDQ4Mw) password：V7ms  
  Next we will take the intallation process of windows version as an example. After downloading, please click the msi file to install. The key installation steps are as follows:
  
  ![IP1](https://github.com/lvyijin/learngit/blob/master/images/first.png "First process")
  
  ![IP2](https://github.com/lvyijin/learngit/blob/master/images/second.png "Second process")
  
  Then click next button according to the default settings. Then the following error occurs: can not find the source file named **common.cab**.
  
  ![IP3](https://github.com/lvyijin/learngit/blob/master/images/problem.png "Problem")
  
  Open the directory where the MSI is located and you will find that the file **common.cab** does not exist, which leads to the failing installation. The solutions are as follows:
  + Please open [link](https://www.oracle.com/technetwork/cn/server-storage/virtualbox/downloads/index.html) and download the installer:
  
    ![installer](https://github.com/lvyijin/learngit/blob/master/images/boxexe.png "virtualBox program")
    
  + Please open cmd and switch to the path of the above installer. Then run the command：
  ```
  VirtualBox-4.3.12-93733-Win.exe -extract
  ```
  + The system prompts it has been extracted to the following directory:
  ```
  c:\users\15627\AppData\Local\Temp\VirtualBox
  ```
   ![extract](https://github.com/lvyijin/learngit/blob/master/images/extract.png "extract")
   
  + Please enter the directory and you can find three files, i.e. **common.cab**, VirtualBox-5.2.8-r121009-MultiArch_amd64.msi (for 64-bit system), VirtualBox-5.2.8-r121009-MultiArch_x86.msi (for 32-bit system).
  + Please click and run VirtualBox-5.2.8-r121009-MultiArch_amd64.msi. Continue to click the next button according to the default settings. After installing successfully, the following icon will be found on the desktop.
  
  ![logo](https://github.com/lvyijin/learngit/blob/master/images/logo.png "logo")
   
## Configuring the experience environment

- Please click the desktop icon to open VirtualBox.
- Click new - Customize name- Select type of **Linux** - Select version of **Ubuntu (64-bit)** - Continue - Select memory of **1024M** - Continue - Select **Existing Virtual Hard Disk File** - Select the **vdi file** - Click create. The whole process is shown below:

  ![instance](https://github.com/lvyijin/learngit/blob/master/images/instance.png "instance")
  
  ![size](https://github.com/lvyijin/learngit/blob/master/images/size.png "size")
  
  ![make](https://github.com/lvyijin/learngit/blob/master/images/make.png "make")
  
- Please start the new instance and set username as ```pouch```, whose password is ```123456```. Run the following command to switch to the root.
```
sudo -i
```
  ![login](https://github.com/lvyijin/learngit/blob/master/images/login.png "login")
  
- Run the command to check whether the network is normal.
```
ping www.alibaba-inc.com
```
It is normal if the outputs are as followings:

  ![ping](https://github.com/lvyijin/learngit/blob/master/images/ping.png "ping")

- Run the command to start the service of pouch.
```
systemctl start pouch
```
- Run the comamnd to start a busybox container.
```
pouch run -t -d busybox sh
```
- Run the comamnd to login in the container.
```
pouch exec -it {ID} sh
```
The ID is the first six digits of the complete ID of the previous command output, as shown below:
  
  ![id](https://github.com/lvyijin/learngit/blob/master/images/id.png "id")
  
Then you will enter the shell of the container. You can run some basic commands, such as ```uname -a``` to prove that you are in the container.
  
## Configuring development environment
### Mounting the host folder to VirtualBox

- From link https://github.com/alibaba/pouch/master to fork the codes to your own github account and clone your master branch to the local.
- Please open the virtualBox and enter the command to view the environment variables of go.
```
go env
```
You will find GOPATH as follows:
```
GOPATH="/root/gopath"
```
If you want to use the host machine for development and run the test in the virtual machine, you should mount the above-mentioned pulled repo to the following path in the virtual machine:
```
/root/gopath/src/github.com/alibaba/pouch
```
- Add the local pouch folder to the shared folder of the virtual machine, as shown below:

![guazai](https://github.com/lvyijin/learngit/blob/master/images/guazai.png "guazai")

![guazai2](https://github.com/lvyijin/learngit/blob/master/images/guazai2.png "guazai2")

- Please create the directory in the virtual machine：
```
mkdir /root/gopath/src/github.com/alibaba/pouch
```
You should ensure that there are no files in the pouch folder, as shown below:

![pouch](https://github.com/lvyijin/learngit/blob/master/images/pouch.png "pouch")

- To mount the shared folder, the enhanced functionality is required, as shown below:

![improve](https://github.com/lvyijin/learngit/blob/master/images/improve.png "improve")

- Please switch to the directory：```/root/gopath/src/github.com/alibaba```, and run the following commands in order.

   ``` 
   mount /dev/cdrom /media/cdrom 
   /media/cdrom/VBoxLinuxAdditions.run 
   mount -t vboxsf pouch /root/gopath/src/github.com/alibaba/pouch
   cd pouch
   ls -l
   ```
   If the outputs are as followings, the mount is successful. Then you can develop happily on the host machine!
![rgua](https://github.com/lvyijin/learngit/blob/master/images/rgua.png "rgua")
![detail](https://github.com/lvyijin/learngit/blob/master/images/detail.png "detail")
# Other tips
## Configuring sshd

Open the virtual machine and configure as shown below:
  
  ![net](https://github.com/lvyijin/learngit/blob/master/images/net.png "net")
  ![port](https://github.com/lvyijin/learngit/blob/master/images/port.png "port")

After configuring the VirtualBox, you need to enable the corresponding ssh port in the virtual machine and allow the root login permission through ssh. Please edit the configuration file in the virtual machine:
```
sudo vi /etc/ssh/sshd_config
```
Confirm that the ssh port is enabled:
```
# What ports, IPs and protocols we listen for
Port 22
```
Allow the root login permission：
```
# Authentication
PermitRootLogin yes
```
Restart ssh when the configuration is completed to make the new configuration take effect:
```
sudo /etc/init.d/ssh restart
```
Then you can login the root account through ssh on the command line of the host:
```
ssh -l root -p 1111 127.0.0.1
```

