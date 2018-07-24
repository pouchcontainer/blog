# 指南说明
由于PouchContainer是企业级容器方案，它只支持Linux操作系统，故需要使用虚拟机才能做到本地运行和测试。本篇指南指导读者在Mac或Windows上搭建基于VirtualBox + Ubuntu16.04的PouchContainer环境。
# 搭建上手步骤
## 1. 安装VirtualBox
1. 在阿里郎安装VirtualBox，或在官网[下载](https://www.virtualbox.org/)VirtualBox安装包：

![image.png | left | 531x345](https://cdn.nlark.com/lark/0/2018/png/121971/1532348887868-0915ef45-e8e0-4fca-9c7a-2d2118ef2434.png "")

2. 安装VirtualBox，以Windows为例，Mac下类似。在安装向导中点击下一步：

![image.png | left | 538x419](https://cdn.nlark.com/lark/0/2018/png/121971/1532349130108-3b9c506d-f652-4add-9914-8c8ea7e926ec.png "")

3. 选择安装功能的方式，这里使用默认值，直接点击两次下一步：

![image.png | left | 537x420](https://cdn.nlark.com/lark/0/2018/png/121971/1532349296176-4b100caa-b402-4f21-aecc-8aff21a4f427.png "")
![image.png | left | 539x421](https://cdn.nlark.com/lark/0/2018/png/121971/1532349417293-6afb7e2b-713f-45ae-bbca-0258af8cc2c9.png "")

4. 此时会警告中断网络，选择是：

![image.png | left | 538x419](https://cdn.nlark.com/lark/0/2018/png/121971/1532349540089-9040e338-406e-401c-8792-5963b557b688.png "")

5. 再确认一次，点击安装：

![image.png | left | 540x419](https://cdn.nlark.com/lark/0/2018/png/121971/1532349617023-ccf1d26e-3aa5-431e-9764-21545b5359bc.png "")

6. 确认安装完成，点击完成：

![image.png | left | 538x422](https://cdn.nlark.com/lark/0/2018/png/121971/1532349733085-bcbf691a-a022-4eaa-96dd-a73c9f02a0b1.png "")

## 2. 配置VirtualBox
1. 在VirtualBox管理器中点击新建，新建虚拟机：

![image.png | left | 536x107](https://cdn.nlark.com/lark/0/2018/png/121971/1532350044579-5118ca49-cc95-4d35-b97f-37064c7b2898.png "")

2. 设置新建虚拟电脑类型版本：

![image.png | left | 500x524](https://cdn.nlark.com/lark/0/2018/png/121971/1532350116607-08386785-f566-47de-a9da-636ad733c2d0.png "")

3. 设置内存大小，这里使用默认值，然后点击下一步：

![image.png | left | 498x523](https://cdn.nlark.com/lark/0/2018/png/121971/1532350251616-d563bc78-41ae-4dec-a55a-861134fd6a31.png "")

4. 选择如何新建虚拟硬盘，这里选择*使用已有的虚拟硬盘文件*，其中的Ubuntu已经安装好了PouchContainer。如果您没有这样的文件，那么可以点击*现在创建虚拟硬盘*，并且，您可能需要自行配置系统，并手动[安装PouchContainer](https://github.com/alibaba/pouch/blob/master/INSTALLATION.md)：

![image.png | left | 502x522](https://cdn.nlark.com/lark/0/2018/png/121971/1532350669919-c7f99d3a-fb3e-4a69-a792-4f16c3657e4c.png "")


## 3. 进入虚拟机，启动并登陆pouch
1. 启动虚拟电脑，登陆，切换到root用户：
```bash
sudo -i
```
2. 检查网络：
```bash
ping www.alibaba-inc.com
```
3. 确认网络没有问题之后，启动pouch服务（默认开机启动）：
```bash
systemctl start pouch
```
4. 启动一个已经存在的busybox基础容器：
```bash
pouch run -t -d busybox sh
```
5. 如果您没有busybox，可以pull：
```bash
pouch pull busybox
```
6. 登录启动的容器，其中ID是启动命令输出的完整ID中的前六位：
```bash
pouch exec -it {ID} sh
```

操作成功后如图所示：

![image.png | left | 747x113](https://cdn.nlark.com/lark/0/2018/png/121971/1532352167171-3167c9b6-9b49-4698-b056-e26073e64312.png "")

至此，您已完成环境搭建，享受PouchContainer吧！
