# PouchContainer environment setup and quick start
##  Related Environment:：

- macOS

- VirtualBox version：5.2.12

  > The links can be found here:：
  >
  > Address of Mac version: https://space.dingtalk.com/s/gwHOABma4QLOGlgkPQPaACBiMzk5ZWRjZTAyOGI0MTBkOGRkNTRjYzNkN2Q1NTFjOA 
  >
  > Password: p5Sb 
  >
  > Address of Windows version: https://space.dingtalk.com/s/gwHOABmLzwLOGlgkPQPaACBhNzNjYjI5NTYxMzQ0NmUwOWRmMTFlN2UzMTYxNDQ4Mw 
  >
  > Password: V7ms 

- CentOS7-Minimal

  > Download link:：
  >
  > https://space.dingtalk.com/s/gwHOABmslALOGlgkPQPaACAwNTg4YTBjOGI4OTI0MGQ5YjE5MDgyYWFjMzAxMDY1MQ Password: tkD3 

- golang version : go1.9.4 linux/amd64

- git version : 1.8.3.1

## Requirements

1. Download VirtualBox and CentOS7-Minimal

2. Open VirtualBox and select New. The specific configuration is as follows:

- Name: Custom

- Type: Linux

- Version selection: RedHat (64bit)

- Memory selection: 1024M


     ![](https://img.alicdn.com/tfs/TB1.wR2DxSYBuNjSspjXXX73VXa-690-442.png)

     ![屏幕快照 2018-07-10 上午2.54.10](https://img.alicdn.com/tfs/TB1yWd8DACWBuNjy0FaXXXUlXXa-564-367.png)

   - Create a virtual hard disk, because virtualbox does not support loading iso files, you need to manually load, select vdi->dynamic allocation->capacity, etc.

     ![屏幕快照 2018-07-10 上午2.59.01](https://img.alicdn.com/tfs/TB1McpNDrSYBuNjSspiXXXNzpXa-566-362.png)

     ![屏幕快照 2018-07-10 上午2.59.07](https://img.alicdn.com/tfs/TB1bClJDA9WBuNjSspeXXaz5VXa-651-429.png)

     ![屏幕快照 2018-07-10 上午2.59.21](https://img.alicdn.com/tfs/TB1xjCnDDlYBeNjSszcXXbwhFXa-652-439.png)

     ![屏幕快照 2018-07-10 上午2.59.37](https://img.alicdn.com/tfs/TB1UqXoDuuSBuNjSsplXXbe8pXa-632-426.png)

3. Start the host after the establishment

   ![屏幕快照 2018-07-10 上午3.03.33](https://img.alicdn.com/tfs/TB1iDXJDA9WBuNjSspeXXaz5VXa-632-514.png)

4. A series of settings for initializing

   ![屏幕快照 2018-07-10 上午3.05.01](https://img.alicdn.com/tfs/TB1eDX2Dv5TBuNjSspmXXaDRVXa-1016-805.png)

   ![屏幕快照 2018-07-10 上午3.06.00](https://img.alicdn.com/tfs/TB1eDGoDrSYBuNjSspfXXcZCpXa-1020-808.png)

   ![屏幕快照 2018-07-10 上午3.07.35](https://img.alicdn.com/tfs/TB1qXVRDx1YBuNjy1zcXXbNcXXa-1019-808.png)

   ![屏幕快照 2018-07-10 上午3.08.21](https://img.alicdn.com/tfs/TB1iwV2DxSYBuNjSspjXXX73VXa-1015-804.png)

   ![屏幕快照 2018-07-10 上午3.15.32](https://img.alicdn.com/tfs/TB1ATXJDA9WBuNjSspeXXaz5VXa-1015-781.png)

   ![屏幕快照 2018-07-10 上午3.16.06](https://img.alicdn.com/tfs/TB1GZ0SDruWBuNjSszgXXb8jVXa-1002-753.png)

   ![屏幕快照 2018-07-10 上午3.16.26](https://img.alicdn.com/tfs/TB1t0b5DDtYBeNjy1XdXXXXyVXa-1011-776.png)

   ![屏幕快照 2018-07-10 上午3.18.39](https://img.alicdn.com/tfs/TB1RcpNDrSYBuNjSspiXXXNzpXa-1014-766.png)

5. Log in as root after rebooting

   ![屏幕快照 2018-07-10 上午3.29.15](https://img.alicdn.com/tfs/TB1ILWRDASWBuNjSszdXXbeSpXa-717-418.png)

6. Configure the network

    ```shell
       cd /etc/sysconfig/network-scripts
       vi ifcfg-enp0s3
       #Enter the network configuration file and edit it
       #modify the onboot option to yes
       #If you have insufficient permissions, you can directly grant permissions.
       #chmod 777 /etc/sysconfig/network-scriptsifcfg-enp0s3
       #Restart the network after the modification is completed.
       service network restart
       #ping www.baidu.com
    ```

   ![屏幕快照 2018-07-10 上午3.30.50](https://img.alicdn.com/tfs/TB1Yp.9Df5TBuNjSspcXXbnGFXa-714-400.png )

   ![屏幕快照 2018-07-10 上午3.41.44](https://img.alicdn.com/tfs/TB1r0xBDrGYBuNjy0FoXXciBFXa-693-37.png )

7. Build go environment：

run:   ```yum install golang```

8. Buil git environment：

run:    ```yum install git```

9. Clone  alibaba/pouch for Github，under the workspace directory

   ```Shell
   #Create a directory (If it hasn't existed)
   mkdir ~/workspace
   #git clone
   git clone https://github.com/alibaba/pouch.git
   ```

10. Configuration of gopath

   ![屏幕快照 2018-07-10 上午4.05.33](https://img.alicdn.com/tfs/TB1YKX7Dr1YBuNjSszeXXablFXa-736-259.png )

11. Install pouch on centos7

12. Install yum utils

    ``` yum install -y yum-utils```

13. Add pouch and update

    ``` yum-config-manager --add-repo http://mirrors.aliyun.com/opsx/opsx-centos7.repo ```

    ``` yum update```

14. Download pouch from mirror of Aliyun 从阿里云的镜像就可以顺利下载pouch服务了

    ``` yum install pouch```

15. Execute the command:systemctl start pouch to start the pouch service.

16. Pull the most basic busybox and start it, then use ls to start it normally.

     ![屏幕快照 2018-07-10 上午4.50.15](https://img.alicdn.com/tfs/TB1DDsrDXmWBuNjSspdXXbugXXa-719-274.png )

17. Successful startup

## 开发环境搭建

In the previous step, we have already built the environment and settings, such as go & git, then you can prepare the solution as follow tips:

- Method 1：Use Vim tools in VirtualBox runtime;

- Method 2：Use share folder in VirualBox, then mount the files into the virual machine.

  1. equipment - share folder - add - mount
  2. mount -t vboxsf \[folder_path\] \[file_name\]
     - e.g. mount -t vboxsf pouch /root/go/src/github.com/alibaba/pouch

  ![屏幕快照 2018-07-10 上午5.16.29](https://img.alicdn.com/tfs/TB1SxvbDv1TBuNjy0FjXXajyXXa-662-366.png )

- Method 3：Using ssh to access virtual machine by using port forward in local machine.
  1. Setting-Network-Gate-Advance-Port forward
  2. Add rules，main port is the localhost，sub port is the virual machine.
     - e.g. main sys port 9022，sub sys port 22
     - run ```ssh -p 9022 root@localhost```

  ![屏幕快照 2018-07-10 上午5.16.22](https://img.alicdn.com/tfs/TB1L_ULDgaTBuNjSszfXXXgfpXa-660-470.png )

- Recommend Solution：

  Method2+Method3，code in the localhost，then use ssh to debug.

## FAQ

### Unable to install guest addition

- Path of iso file: /Applications/VirtualBox.app/Contents/MacOS/VBoxGuestAdditions.iso
- Open VirtualBox, open Machine - Settings - Storage, and add a virtual CD in the controller

![屏幕快照 2018-07-08 下午11.02.48](https://img.alicdn.com/tfs/TB1T4lvDpuWBuNjSszbXXcS7FXa-269-30.png)

![屏幕快照 2018-07-08 下午11.02.52](https://img.alicdn.com/tfs/TB1XaBRDER1BeNjy0FmXXb0wVXa-362-146.png)

- Start the virtual machine, and the CD-ROM should be now under /dev/cdrom. Now use `mount` command to mount the CD-ROM to mnt.

```
  mkdir mount/cdrom
  mount /dev/cdrom /mnt/cdrom
```

- Enter the directory (cd /mnt/cdrom) and execute the corresponding script: `sh VBoxLinuxAdditions.run`.
- Some packages might be missing. If so, use `yum install kernel-devel` to install relevent packages and re-execute the previous step.
- Complete installation of guest addition.

