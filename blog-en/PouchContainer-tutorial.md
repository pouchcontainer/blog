## Introduction of PouchContainer
PouchContainer is an open-source container engine technology in enterprise level created by Alibaba. It can provide strong isolation with high protability and occupy limited resource at the same time. PouchContainer can not only help enterprises achieve containerization of their stock business,but also improve the physical resource usage of data center in large scale.

In this passage, the installation and a tutorial of PouchContainer will be introduced.


## Prepare Environment

- VirtualBox

- Ubuntu image with pouch

  

## Building steps

1. Open the already installed VirtualBox. First, click the button NEW on the menu to create a new operation system. Configure as below: name can be defined freely; choose type as Linux; choose Version as Red Hat(64-bit).

   ![](http://oyrpkn4bk.bkt.clouddn.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202018-07-30%20%E4%B8%8B%E5%8D%885.33.31.png)

   

2. Click NEXT, entering the page with RAM otpions. Choose RAM as 1024 MB. You can choose larger RAM regarding to your need. 

   ![](http://oyrpkn4bk.bkt.clouddn.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202018-07-30%20%E4%B8%8B%E5%8D%885.25.41.png)

   

3. Click NEXT, engtering the page with hard disk options. Choose 'Using existing virtual hard disk file', then find the Ubuntu Image containing pouch `ubuntuPouch.vdi`. Click create. Finally a new created virtual machine is showed on the left bar.

   ![](http://oyrpkn4bk.bkt.clouddn.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202018-07-30%20%E4%B8%8B%E5%8D%885.34.02.png)

   

4. Activate virtual machine, wating for the log in message. Enter the user name `pouch` and the code`123456`

5. Switch to the root user：`sudo su root`

6. Check Internet：`ping www.alibaba-inc.com`

7. Activate pouch：`systemctl start pouch`

8. Type `pouch` order to see whether it has been activated

   ![](http://oyrpkn4bk.bkt.clouddn.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202018-07-30%20%E4%B8%8B%E5%8D%885.43.46.png)

   

9.  Execute `pouch run -t -d busybox sh` to start a container named as busybox

10. Execute `pouch ps -a` to see all of the containers that has been created 

    ![](http://oyrpkn4bk.bkt.clouddn.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202018-07-30%20%E4%B8%8B%E5%8D%886.05.59.png)

    

11. Execute `pouch exec -it {id} sh` to enter the container where {id} is the first six characters of the result got from the last command.

![](http://oyrpkn4bk.bkt.clouddn.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202018-07-30%20%E4%B8%8B%E5%8D%886.06.23.png)  

So far, PouchContainer has been installed in the Ubuntu virtual machine. You can follow the same instruction to set up Centos, since the installation process of Centos is similar.  

For the conveinence of local program development, we will show how to configure the ssh connection of the virtual machine.




## Connecting virtual machine via SSH

1. In the interface of VirtualBox, click Settings -> Network. Check ‘Enable Network Adapter’. Choose 'NAT' in 'Attached to' options.

2. Click Advance -> Port Forwarding. Then you can configure the ip address and port number of host and guest. The Host Port  can be chosen randomly while the Guest Port must be 22. 

   

   ![](http://oyrpkn4bk.bkt.clouddn.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202018-07-30%20%E4%B8%8B%E5%8D%885.56.06.png)

   

3. After configuration，open local terminal，type command `ssh -p 2233 pouch@127.0.0.1` to connect to the virtual machine.

   ![](http://oyrpkn4bk.bkt.clouddn.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202018-07-30%20%E4%B8%8B%E5%8D%885.58.24.png)



## Conclusion

Basing on the tutorial showed above, we could use PouchContainer easily in any non-Linux computer.







