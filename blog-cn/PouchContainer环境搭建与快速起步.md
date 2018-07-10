# PouchContainer环境搭建与快速起步

## 相关环境：

- macOS

- VirtualBox version：5.2.12

  > 参考下载链接：
  >
  > Mac 版本地址: https://space.dingtalk.com/s/gwHOABma4QLOGlgkPQPaACBiMzk5ZWRjZTAyOGI0MTBkOGRkNTRjYzNkN2Q1NTFjOA 
  >
  > 密码: p5Sb 
  >
  > Windows 版本地址: https://space.dingtalk.com/s/gwHOABmLzwLOGlgkPQPaACBhNzNjYjI5NTYxMzQ0NmUwOWRmMTFlN2UzMTYxNDQ4Mw 
  >
  > 密码: V7ms 

- CentOS7-Minimal

  > 参考下载链接：
  >
  > https://space.dingtalk.com/s/gwHOABmslALOGlgkPQPaACAwNTg4YTBjOGI4OTI0MGQ5YjE5MDgyYWFjMzAxMDY1MQ 密码: tkD3 

- golang version : go1.9.4 linux/amd64

- git version : 1.8.3.1

## 体验环境搭建

1. 首先下载VirtualBox以及CentOS7-Minimal

2. 打开VirtualBox，选择新建，具体配置参考如下：

   - 名称：自定义

   - 类型：Linux

   - 版本选择：RedHat(64bit)

   - 内存选择：1024M

     ![](https://img.alicdn.com/tfs/TB1.wR2DxSYBuNjSspjXXX73VXa-690-442.png)

     ![1](https://img.alicdn.com/tfs/TB1yWd8DACWBuNjy0FaXXXUlXXa-564-367.png)

   - 创建虚拟硬盘，在此处因为virtualbox不支持加载iso文件的缘故需要手动load，依次选择vdi->动态分配->容量等

     ![2](https://img.alicdn.com/tfs/TB1McpNDrSYBuNjSspiXXXNzpXa-566-362.png)

     ![3](https://img.alicdn.com/tfs/TB1bClJDA9WBuNjSspeXXaz5VXa-651-429.png)

     ![4](https://img.alicdn.com/tfs/TB1xjCnDDlYBeNjSszcXXbwhFXa-652-439.png)

     ![5](https://img.alicdn.com/tfs/TB1UqXoDuuSBuNjSsplXXbe8pXa-632-426.png)

3. 建立完毕启动主机

   ![6](https://img.alicdn.com/tfs/TB1iDXJDA9WBuNjSspeXXaz5VXa-632-514.png)

4. 进行初始化的一系列设置

   ![7](https://img.alicdn.com/tfs/TB1eDX2Dv5TBuNjSspmXXaDRVXa-1016-805.png)

   ![8](https://img.alicdn.com/tfs/TB1eDGoDrSYBuNjSspfXXcZCpXa-1020-808.png)

   ![9](https://img.alicdn.com/tfs/TB1qXVRDx1YBuNjy1zcXXbNcXXa-1019-808.png)

   ![11](https://img.alicdn.com/tfs/TB1iwV2DxSYBuNjSspjXXX73VXa-1015-804.png)

   ![](https://img.alicdn.com/tfs/TB1ATXJDA9WBuNjSspeXXaz5VXa-1015-781.png)

   ![](https://img.alicdn.com/tfs/TB1GZ0SDruWBuNjSszgXXb8jVXa-1002-753.png)

   ![](https://img.alicdn.com/tfs/TB1t0b5DDtYBeNjy1XdXXXXyVXa-1011-776.png)

   ![](https://img.alicdn.com/tfs/TB1RcpNDrSYBuNjSspiXXXNzpXa-1014-766.png)

5. 重启后以root用户登录

   ![](https://img.alicdn.com/tfs/TB1ILWRDASWBuNjSszdXXbeSpXa-717-418.png)

6. 配置网络

    ```shell
       cd /etc/sysconfig/network-scripts
       vi ifcfg-enp0s3
       #进入网络配置文件并进行编辑
       #如下图，修改onboot选项为yes
       #如遇权限不够的情况直接赋予权限
       #chmod 777 /etc/sysconfig/network-scriptsifcfg-enp0s3
       #修改完成后重启网络
       service network restart
       #ping www.baidu.com 成功即可
    ```

   ![](https://img.alicdn.com/tfs/TB1Yp.9Df5TBuNjSspcXXbnGFXa-714-400.png )

   ![](https://img.alicdn.com/tfs/TB1r0xBDrGYBuNjy0FoXXciBFXa-693-37.png )

7. 进行go环境的搭建：

   执行```yum install golang```

8. 进行git环境搭建：

   执行```yum install git```

9. 完毕后从GitHub仓库上clone  alibaba/pouch项目，项目需要clone在workspace目录下

   ```Shell
   #创建目录 如有则无需创建
   mkdir ~/workspace
   #git clone
   git clone https://github.com/alibaba/pouch.git
   ```

10. 进行gopath配置

   ![](https://img.alicdn.com/tfs/TB1YKX7Dr1YBuNjSszeXXablFXa-736-259.png )

11. 接下来是在centos7上pouch服务的安装

12. 安装yum utils

    ``` yum install -y yum-utils```

13. 添加pouch仓库并update

    ``` yum-config-manager --add-repo http://mirrors.aliyun.com/opsx/opsx-centos7.repo ```

    ``` yum update```

14. 从阿里云的镜像就可以顺利下载pouch服务了

    ``` yum install pouch```

15. 执行命令 systemctl start pouch 启动 pouch 服务 

16. pull下最基本的busybox并启动，在里面正常使用ls即启动完成

     ![](https://img.alicdn.com/tfs/TB1DDsrDXmWBuNjSspdXXbugXXa-719-274.png )

17. 启动成功

## 开发环境搭建

在上一步中我们也已经做好了初步的开发环境搭建，go/git等工具的配置，接下来提供一些开发环境的解决方案

- 方案一：在VirtualBox虚拟机中直接进行开发，使用vim等进行代码编写

- 方案二：使用VirtualBox的共享文件夹功能，进行宿主机向虚拟机的文件挂载

  1. 设备-共享文件夹-添加-固定分配&自动挂载
  2. 保存之后执行挂载命令
  3. mount -t vboxsf \[文件夹名字\] \[具体路径\]
     - e.g. mount -t vboxsf pouch /root/go/src/github.com/alibaba/pouch

  ![](https://img.alicdn.com/tfs/TB1SxvbDv1TBuNjy0FjXXajyXXa-662-366.png )

- 方案三：进行端口映射后在宿主机使用terminal等工具使用ssh服务登陆虚拟机进行开发

  1. 设置-网络-网卡1-高级-端口转发
  2. 添加规则，主机端口即宿主机访问端口，子系统端口即虚拟机访问端口
     - e.g. 主机端口9022，子系统端口22
     - terminal中访问```ssh -p 9022 root@localhost```即可

  ![](https://img.alicdn.com/tfs/TB1L_ULDgaTBuNjSszfXXXgfpXa-660-470.png )

- 最佳实践：

  方案二+方案三，在宿主机上进行代码的编写，然后通过ssh服务进入虚拟机进行调试

## 可能出现的相关问题解决

### guest addition无法安装问题 

- iso文件路径	：/Applications/VirtualBox.app/Contents/MacOS/VBoxGuestAdditions.iso
- 打开VirtualBox－设置－存储，在控制器中添加虚拟光盘

![](https://img.alicdn.com/tfs/TB1T4lvDpuWBuNjSszbXXcS7FXa-269-30.png)

![](https://img.alicdn.com/tfs/TB1XaBRDER1BeNjy0FmXXb0wVXa-362-146.png)

- 进入虚拟机，挂载的光盘实例为/dev/cdrom，使用mount命令挂载到mnt上

```
  mkdir mount/cdrom
  mount /dev/cdrom /mnt/cdrom
```

- cd /mnt/cdrom 进入目录 执行对应脚本，sh VBoxLinuxAdditions.run
- 可能缺失一些包，yum install kernel-devel安装，然后重新执行上一步
- 完成guest addition安装