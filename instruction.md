# Set Up Enviroment

Since PouchContainer is an enterprise container solution, it only supports Linux operating system. In order to use PouchContainer, we use VirtualBox+CentOS7 to set up environment. 

1.Download and install virtualBox and image of CentOS7.  

2.Start virtualBox and new a VM.  

![avator](https://img.alicdn.com/tfs/TB1V7NzDx9YBuNjy0FfXXXIsVXa-720-424.png)

3.The name of VM can be **customized**; Select type **Linux**; Select version **Red Hat(64-bit)**.  

![avator](https://img.alicdn.com/tfs/TB1upE.DXOWBuNjy0FiXXXFxVXa-572-370.png)

4.Select memory size **1024M**.  

![avator](https://img.alicdn.com/tfs/TB1hORzDx9YBuNjy0FfXXXIsVXa-572-370.png)

5.Select **Using an exsiting virtual hard disk file**.  

![avator](https://img.alicdn.com/tfs/TB194I4DbGYBuNjy0FoXXciBFXa-572-370.png)

6.Start the VM, and login as root.

![avator](https://img.alicdn.com/tfs/TB1I_fuDDtYBeNjy1XdXXXXyVXa-720-443.png) 

7.Use vim to modify the file /etc/sysconfig/network-scripts/ifcfg-eth0. Make sure HWADDR in the file is consistent with the Mac address shown after running ip ad in terminal.
Then reboot the VM and ping ip to make sure the network configuration is correct. So far, the system environment has been set up. Let's install PouchContainer.

# Installation And Startup  

You can install PouchContainer automatically on your machine with very few steps. Currently we support two kinds of Linux Distribution: Ubuntu and CentOS.  

1.Install yum-utils  
Install required packages. yum-utils provides the yum-config-manager utility.     

```
sudo yum install -y yum-utils
```

2.Set up the PouchContainer repository   

```
sudo yum-config-manager --add-repo http://mirrors.aliyun.com/opsx/opsx-centos7.repo
sudo yum update
```

3.Install PouchContainer 

```
sudo yum install pouch
```

4.Start PouchContainer

```
sudo systemctl start pouch
```

# Create instance
1.start service

```
systemctl start pouch
```

2.Start a basic container of busybox

```
pouch run -t -d busybox sh
```

3.Login the started container

```
pouch exec -it {ID} sh
```

The ID is the first six digits of the previous command's output.
