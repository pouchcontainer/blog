# 基于 VirtualBox + Ubuntu16.04 的 PouchContainer 体验环境搭建与上手指南

## 下载VirtualBox

#### 1. 阿里郎-管家-办公软件管理 安装 VirtualBox，默认版本为 5.2.12，若没有安装阿里郎，请参考步骤 2 在钉盘下载
#### 2. 在钉盘上下载 VirtualBox

- Mac 版本地址: [Mac版本VirtualBox下载](https://space.dingtalk.com/s/gwHOABma4QLOGlgkPQPaACBiMzk5ZWRjZTAyOGI0MTBkOGRkNTRjYzNkN2Q1NTFjOA)   密码：p5Sb

- Windows 版本地址: [Windows版本VirtualBox下载](https://space.dingtalk.com/s/gwHOABmLzwLOGlgkPQPaACBhNzNjYjI5NTYxMzQ0NmUwOWRmMTFlN2UzMTYxNDQ4Mw)   密码：V7ms

## 配置虚拟机

#### 1. 下载开发环境的虚拟机备份

- 群文件中下载ubuntuPouch.vdi

#### 2. 创建新虚拟机

- 打开VirtualBox，选择新建 -> 名称自定义 -> 类型选择【Linux】-> 版本选择【Ubuntu(64-bit)】-> 单击继续

![图1.png](https://cdn.nlark.com/lark/0/2018/png/118013/1532345730447-9ab4831f-dd14-403d-b183-352ac9f4f34d.png)

- 内存选择【1024M】-> 单击继续

![图2.png](https://cdn.nlark.com/lark/0/2018/png/118013/1532346452738-056807ee-1e09-44fe-a796-def919ddb104.png)

- 使用【已有的虚拟硬盘文件】-> 选择步骤 1 中下载的 vdi 文件 -> 单击创建

![图3.png](https://cdn.nlark.com/lark/0/2018/png/118013/1532346553517-6ed9bbbe-0473-477f-b23b-13aa9a14e889.png)

#### 3. 启动新建实例，等待进入到登录阶段，用户名 pouch ，密码 123456
![图4.png](https://cdn.nlark.com/lark/0/2018/png/118013/1532346622257-fd31e9f0-c12a-4ff5-865c-858351c81731.png)

#### 4.切换到root用户

- 虚拟机终端输入 sudo -i
- 输入pouch用户的密码

![图5.png](https://cdn.nlark.com/lark/0/2018/png/118013/1532347359233-0ec1fae6-75f4-4c26-9816-ae73bd8ea5e7.png)

#### 5. 检查网络是否正常

- ping www.alibaba-inc.com ,检查网络是否正常

![图6.png](https://cdn.nlark.com/lark/0/2018/png/118013/1532347369091-44e1ce1f-1a95-4d47-8f46-e245441137d2.png)

## 运行PouchContainer

#### 1. 启动pouch服务

- 在虚拟机终端执行命令 systemctl start pouch 启动pouch服务（默认开机启动）

#### 2.启动busybox基础容器

- 执行 pouch run -t -d busybox sh 启动一个busybox基础容器，该命令的输出为容器ID

#### 3.登入启动的容器

- 执行 pouch exec -it {ID} sh 登入启动的容器，其中ID是上步骤 2 中命令输出的完整ID的前六位

![图7.png](https://cdn.nlark.com/lark/0/2018/png/118013/1532348025526-370f1c96-1f02-4498-a005-4aed4261114c.png)