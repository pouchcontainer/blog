# 基于 VirtualBox + CentOS7 的 PouchContainer 体验环境搭建与上手指南 

## 下载VirtualBox

**1. 阿里郎-管家-办公软件管理 安装 VirtualBox，默认版本为 5.2.12**

**2. 在钉盘上下载 VirtualBox**

- Mac 版本地址: https://space.dingtalk.com/s/gwHOABma4QLOGlgkPQPaACBiMzk5ZWRjZTAyOGI0MTBkOGRkNTRjYzNkN2Q1NTFjOA
- Windows 版本地址: https://space.dingtalk.com/s/gwHOABmLzwLOGlgkPQPaACBhNzNjYjI5NTYxMzQ0NmUwOWRmMTFlN2UzMTYxNDQ4Mw

## 配置虚拟机

**1. 下载开发环境的虚拟机备份，钉盘地址:** 

https://space.dingtalk.com/s/gwHOABmLxALOGlgkPQPaACA4N2JjNmIwMGI5NWU0MGE1YjZhNTBiOGNjMDZhOTJiNQ

**2. 打开 VirtualBox**

- 新建-名称自定义-类型选择【Linux】-版本选择【Red Hat(64- bit)】 

<img src="https://cdn.yuque.com/lark/0/2018/png/72820/1531142367376-58af5f86-e7e1-452e-aee3-cce1489a1a92.png" width="450px" height="300px" />

- 内存选择【1024M】

<img src="https://cdn.yuque.com/lark/0/2018/png/72820/1531142356237-1525281e-6fbd-4dfa-b359-8f10f9720ef6.png" width="450px" height="300px" />

- 使用【已有的虚拟硬盘文件】-选择步骤 1 中下载的 vdi 文件-创建

<img src="https://cdn.yuque.com/lark/0/2018/png/72820/1531142341940-d29a6743-c8e5-4581-adda-4e4f1149f1d2.png" width="450px" height="300px" />

**3. 启动新建实例，等待进入到登录阶段，用户名 `root`**

<img src="https://cdn.yuque.com/lark/0/2018/png/72820/1531142322664-3227342d-fa41-48b3-834e-3aa46b8446a2.png" width="700px" height="150px" />

**4. 修改`/etc/sysconfig/network-scripts/ifcfg-eth0`，使其中的 HWADDR 与 ip ad 命令中显示的 MAC 地址一致，reboot 后 ping  www.alibaba-inc.com，检查网络是否正常**

-  ip ad 命令查看MAC地址

<img src="https://cdn.yuque.com/lark/0/2018/png/72820/1531142281019-3a911e83-76fb-47e9-b11e-3d97a960fbd7.png" width="700px" height="400px" />

- 修改`/etc/sysconfig/network-scripts/ifcfg-eth0`中HWADDR

<img src="https://cdn.yuque.com/lark/0/2018/png/72820/1531142305340-0966e48f-2bdc-492a-8277-791a92c62e0f.png" width="700px" height="130px" />

- ping www.alibaba-inc.com，检查网络连接

<img src="https://cdn.yuque.com/lark/0/2018/png/72820/1531142294988-03c54bb0-6778-4fb1-b149-9f5fa3e1122e.png" width="700px" height="200px" />

## 运行PouchContainer

1. 执行 `systemctl start pouch`， 启动 pouch 服务 
2. 执行 `pouch run -t -d busybox sh`， 启动一个 busybox 基础容器 
3. 执行 `pouch exec -it {ID} sh` 登入启动的容器，其中 ID 是上条命令输出的完整 ID 中的前六位 

<img src="https://cdn.yuque.com/lark/0/2018/png/72820/1531142287443-e03b22c0-06d9-48ab-a13f-ea8b6718877d.png" width="700px" height="100px" />

## 配置sshd

**1. 在VirtualBox中设置网卡，选择仅主机（Host-Only）适配器的方式**

<img src="https://cdn.yuque.com/lark/0/2018/png/72820/1531142079098-7c0cc1d6-3d78-4194-8d44-5cbc6e7ff2bb.png" width="450px" height="250px" />

**2. 在虚拟机中修改sshd配置，允许远程连接**

<img src="https://cdn.yuque.com/lark/0/2018/png/72820/1531142261139-1571f8f5-cc5a-4610-963c-b4930a16b69c.png" width="550px" height="300px" />

<img src="https://cdn.yuque.com/lark/0/2018/png/72820/1531143160892-bf7a520c-fc96-40b6-a9bf-d4286c9673ca.png" width="550px" height="300px" />

**3. 重启sshd服务：`systemctl restart sshd.service `**

**4. 查看虚拟机IP地址：`ip addr `**

<img src="https://cdn.yuque.com/lark/0/2018/png/72820/1531142232272-1d4b4cb6-623a-4336-8b41-e8568458814d.png" width="450px" height="300px" />

**5. 在宿主机中通过iTerm连接虚拟机**

 <img src="https://cdn.yuque.com/lark/0/2018/png/72820/1531143146304-720f9ef3-1031-4b73-aa60-250c44c41c26.png" width="700px" height="150px" />

## 将宿主机pouch目录挂载到虚拟机中 

**1. 在VirtualBox中设置共享文件夹**

<img src="https://cdn.yuque.com/lark/0/2018/png/72820/1531142079098-7c0cc1d6-3d78-4194-8d44-5cbc6e7ff2bb.png" width="500px" height="300px" />

**2. 在虚拟机中执行以下命令安装挂载所需模块**

```
yum clean all
yum update
yum install kernel 
yum install kernel-devel 
yum install kernel-headers 
yum install gcc 
yum install make   
reboot 
cd /opt/VBoxGuestAdditions-*/init  
./vboxadd setup 
reboot
```

**3. 执行` mount -t vboxsf pouchShare /root/go/src/github.com/alibaba/pouch`，将添加的共享文件夹挂载到虚拟机的pouch源码路径下**

