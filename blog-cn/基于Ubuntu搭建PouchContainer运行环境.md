## PouchContainer简介

PouchContainer是阿里巴巴集团开源的高效、企业级容器引擎技术，拥有隔离性强、可移植性高、资源占用少等特点。可以帮助企业快速实现存量业务容器化，同时提高超大规模下数据中心的物理资源利用率。

本文将给大家介绍PouchContainer的搭建和上手过程。



## 环境准备

- VirtualBox

- 包含pouch的Ubuntu镜像

  

## 搭建过程

1. 打开安装好的VirtualBox，首先点击标题栏的New按钮，新建一个操作系统，name可以自定义，type选择Linux，Version选择Red Hat（64-bit)。

   ![](http://oyrpkn4bk.bkt.clouddn.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202018-07-30%20%E4%B8%8B%E5%8D%885.33.31.png)

   

2. 点击继续按钮进入内存的选择页面。内存选择1024MB，当然也可以根据需要加大内存。

   ![](http://oyrpkn4bk.bkt.clouddn.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202018-07-30%20%E4%B8%8B%E5%8D%885.25.41.png)

   

3. 点击继续按钮进入硬盘选择页面，选中"使用已有的虚拟硬盘文件"，选择包含pouch的Ubuntu镜像`ubuntuPouch.vdi`，点击创建，在VirtualBox左边栏可以看到新建的虚拟机。

   ![](http://oyrpkn4bk.bkt.clouddn.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202018-07-30%20%E4%B8%8B%E5%8D%885.34.02.png)

   

4. 启动虚拟机，等待进入登录阶段，用户名`pouch`，密码`123456`

5. 切换到root用户下：`sudo su root`

6. 检查网络是否通畅：`ping www.alibaba-inc.com`

7. 启动pouch服务：`systemctl start pouch`

8. 输入`pouch`命令判断pouch是否启动

   ![](http://oyrpkn4bk.bkt.clouddn.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202018-07-30%20%E4%B8%8B%E5%8D%885.43.46.png)

   

9. 执行`pouch run -t -d busybox sh`启动名为busybox的容器

10. 使用`pouch ps -a`可以查看所有创建的容器

    ![](http://oyrpkn4bk.bkt.clouddn.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202018-07-30%20%E4%B8%8B%E5%8D%886.05.59.png)

    

11. 执行`pouch exec -it {id} sh`进入容器

![](http://oyrpkn4bk.bkt.clouddn.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202018-07-30%20%E4%B8%8B%E5%8D%886.06.23.png)  

到此为止，PouchContainer在Ubuntu虚拟机上的搭建过程已经完成，Centos与此类似。

为了本地开发方便，我们将继续配置虚拟机的ssh连接。



## SSH连接虚拟机

1. 在VirtualBox界面点击设置-->网络，连接方式选择网络地址转换

2. 在该界面下点击高级，端口转换，配置端口映射。主机端口可以任选，子系统端口必须选择22。

   

   ![](http://oyrpkn4bk.bkt.clouddn.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202018-07-30%20%E4%B8%8B%E5%8D%885.56.06.png)

   

3. 配置完成，打开本地terminal，使用命令`ssh -p 2233 pouch@127.0.0.1`连接虚拟机。

   ![](http://oyrpkn4bk.bkt.clouddn.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202018-07-30%20%E4%B8%8B%E5%8D%885.58.24.png)



## 总结

通过上面的教程，我们可以很轻松的在非Linux电脑上体验PouchContainer。







