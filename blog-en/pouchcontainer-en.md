# How to build a running environment for PouchContainer?

## About Pouch Container

PouchContainer is Alibaba Group's open source, efficient, and lightweight enterprise container, which helps enterprises quickly improve server utilization efficiency. PouchContainer has been used in Alibaba for many years with strong reliability and stability.

This guide will instruct you how to quickly build a running environment in Windows/Mac for PouchContainer using Virtual Box and Ubuntu16.04 so that users may use PouchContainer in other operation systems.

## Prerequisite

Before you begin, download and install VirtualBox and UbuntuPouch

### VirtualBox

#### To download VirtualBox

Choose one of the following ways:

- From Dingtalk Cloud
	
	MacVersion£º[Mac Download](https://space.dingtalk.com/s/gwHOABma4QLOGlgkPQPaACBiMzk5ZWRjZTAyOGI0MTBkOGRkNTRjYzNkN2Q1NTFjOA)    Password: p5Sb

	Windows Version£º[Windows Download](https://space.dingtalk.com/s/gwHOABmLzwLOGlgkPQPaACBhNzNjYjI5NTYxMzQ0NmUwOWRmMTFlN2UzMTYxNDQ4Mw) Password: V7ms
   
- From Alilang
    
	1. Open Alilang.
	
	2. On the left navigation bar, choose **Management**.
	
	3. On the right page, click **Office Software Management**.
    
	4. On the search bar, put **VirtualBox**.
	
	5. Click **Download**.

####To Install VirtualBox

Practice the following steps:

1. Double click the **Oracle VM VirtualBox 5.2.12** package downloaded.

2. Click **Run**.

3. Click **Next**.

4. Click **Next**.

5. Click **Next**.

6. Click **Yes**.

7. Click **Install**.

  **Note:** when installing VirtualBox, a box of Oracle Corporation shows. Click **Install** .

8. After installing completes, click **Finish**.

### UbuntuPouch

#### To download UbuntuPouch

Practice the following steps:

1. Open Dingtalk.

2. On the left bar, find the group.

3. On the right dialogue box, click **file**.

4. On the right navigation bar, click **UbuntuPouch.vdi**.

5. Click **Download**.


#### To Install UbuntuPouch into VirtualBox

Practice the following steps:

1. Open VirtualBox.

2. On the menu, click **New**.

3. On the name of the box, put self-defined name \(for example, Pouch)\.

4. Click **Type**, choose **Linux**.

5. Click **Version**, choose **Ubuntu \(64-bit)\**.

6. Click **Next**.

7. Click **Memory**, choose **1024M**.

8. Click **Next**.

9. Choose **Virtual Disk File Already Existed**.

10. Add file path and choose the UbuntuPouch you downloaded.

11.Click **Open**.

12. Click **Create**.
 
 
## Steps: to create a pouchcontainer instance

After meet the prerequisite, practice the following steps:

1. Double click the self-defined pouch.

2. Put `pouch` and press enter.

3. Put `123456` and press enter.

4. Put `su` and press enter.

5. Put `1234566` and press enter.

  **Note:** If it does notjump to root account fails, reset your password and the command is `sudo passwd root`. Put your password twice.

6. Put `ping www.alibaba-inc.com` and press enter, check if the network works.

7. Put `cd /root/gopath/src/github.com/alibaba/pouch` and press enter.

8. Put `rm -rf ./* `and press enter.

9. Put `git config --global user.name "{Account name}"` and press enter.

10. Put `git config --global user.email "{email address}"` and press enter.

11. Put `git init` and press enter.

12. Put `git clone {github project URL}` and press enter.

13. Put `ps -ef | grep pouch` and press enter, check if the pouch starts.

  **Note:** If pouch fails, put `systemctl start pouch` and press enter.

14. Put `Pouch run -t -d busybox sh` and press enter to start a busybox basic container.

15. Put `Pouch exec -it {ID} sh` and press enter to log on the container, the first six digits of the ID of the last command is shown
