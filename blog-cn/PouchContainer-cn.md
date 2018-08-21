# 如何为 PouchContainer 搭建运行环境?

## 背景介绍

PouchContainer 是阿里巴巴集团开源的高效、轻量级企业级富容器引擎技术，可以帮助企业快速提升服务器的利用效率。PouchContainer 在阿里内部经过多年的使用，已经具有较强的可靠性和稳定性。目前 PouchContainer 仅支持在 Ubuntu 和 CentOS 上运行。

下面将介绍如何在 Windows/Mac 中通过 VirtualBox 和 Ubuntu16.04 快速搭建 PouchContainer 的运行环境，帮助用户在其他的操作系统中也可以使用 PouchContainer。 

## 前提条件

在您开始搭建工作前，请先下载并安装 VirtualBox 和 UbuntuPouch。

### VirtualBox

#### 下载 VirtualBox

选择以下两种方式中的任意一种:

- 钉盘下载

Mac 版本：[Mac Download](https://space.dingtalk.com/s/gwHOABma4QLOGlgkPQPaACBiMzk5ZWRjZTAyOGI0MTBkOGRkNTRjYzNkN2Q1NTFjOA)    密码: p5Sb

Windows 版本：[Windows Download](https://space.dingtalk.com/s/gwHOABmLzwLOGlgkPQPaACBhNzNjYjI5NTYxMzQ0NmUwOWRmMTFlN2UzMTYxNDQ4Mw) 密码：V7ms

- 阿里郎下载

1. 打开阿里郎；

2. 在左侧导航栏，选择**管家**；

3. 在右侧页面，选择**办公软件管理**；

4. 在右上方的搜索栏, 输入 **VirtualBox**；

5. 单击 **安装**。

####安装 VirtualBox

请参考以下步骤执行：

1. 双击下载到本地的 **Oracle VM VirtualBox 5.2.12** 安装包；

2. 单击**运行**；

3. 单击**下一步**；

4. 单击**下一步**；

5. 单击**下一步**；

6. 单击**是**；

7. 单击**安装**；

**注意:** 安装 VirtualBox 过程中，如果弹出是否安装 Oracle Corporation 的窗口，请单击 **安装**。 

8. 安装结束后，单击**完成**。

### UbuntuPouch

#### 下载 UbuntuPouch

请参考以下步骤执行：

1. 打开钉钉；

2. 在左侧栏，找到百技群；

3. 在右侧对话框 ，单击**文件**；

4. 在右侧导航栏，单击 **UbuntuPouch.vdi**；

5. 单击**下载**。


#### 将 UbuntuPouch 安装到 VirtualBox

请参考以下步骤执行：

1. 打开 VirtualBox；

2. 在菜单栏，选择**新建**；

3. 在**名称**一栏 ，输入自定义的名称（例如，pouch）；

4. 单击**类型**一栏，选择 **Linux**；

5. 单击**版本**，选择 **Ubuntu \(64-bit)\**；

6. 单击**下一步**；

7. 将**内存大小**调整为**1024MB**（默认）；

8. 单击**下一步**；

9. 选择**使用已有的虚拟硬盘文件**；

10. 添加文件路径，选择之前下载到本地的 UbuntuPouch；

11. 单击**打开**；

12. 单击**创建**。

## 步骤：创建一个 pouchcontainer 实例

在完成准备工作后，请参考以下步骤执行：

1. 在 VirtualBox 中，双击自定义的 pouch；

2. 输入 `pouch`并按回车键；

3. 输入`123456`并回车键；

4. 输入 `su`并回车键；

5. 输入`1234566`并按回车键；

**注意：** 如果未跳转到根目录，请重新设置您的密码（命令行：`sudo passwd root`）。请输入您的密码两次。

6. 输入 `ping www.alibaba-inc.com` 并按回车键， 检查网络是否正常；

7. 输入 `cd /root/gopath/src/github.com/alibaba/pouch` 并按回车键；

8. 输入 `rm -rf ./* `并按回车键；

9. 输入 `git config --global user.name "{账户名称}"` 并按回车键；

10. 输入 `git config --global user.email "{邮箱地址}"` 并按回车键；

11. 输入 `git init` 并按回车键;

12. 输入 `git clone {Gitub 项目 URL}` 并按回车键；

13. 输入 `ps -ef | grep pouch` 并按回车键，检查 pouch 是否启动；

**注意：** 如果 pouch 启动失败，输入 `systemctl start pouch` 并按回车键。

14. 输入 `Pouch run -t -d busybox sh` 并按回车键，启动一个 busybox basic container

15. 输入 `Pouch exec -it {ID} sh` 并按回车键，登录这个 container，将会显示上个命令的 ID 的前六位数字。
