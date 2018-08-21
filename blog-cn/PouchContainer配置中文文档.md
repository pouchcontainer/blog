## PouchContainer简介

PouchContainer是阿里巴巴集团为了促进容器技术（container）的发展而建立的开源项目，它具有量级轻、隔离性强、可移植性高、资源占用少等特性，可以帮助企业快速实现业务容器化，同时提高超大规模下数据中心的物理资源利用率。

## GitHub用户配置及源码下载

* 配置GitHub用户信息。

  ```bash
  git config --global user.name "<your_github_username>"  # set Github username
  git config --global user.email "<your_github_email>"  # set Github email
  git config --global credential.helper store  # avoid typing password every time during pulling
  ```

* 在浏览器中打开pouch的源码repo：<https://github.com/alibaba/pouch>.

* 点击右上角Fork，完成Fork操作。

* 在本机的终端clone代码。

  ```bash
  git clone https://github.com/<your_github_username>/pouch.git
  ```

## 创建并启动Ubuntu虚拟机

* 安装VirtualBox。使用<阿里郎>--<管家>--<办公软件管理>安装，默认版本为5.2.12。钉盘链接：
  * Mac版本：<https://space.dingtalk.com/s/gwHOABma4QLOGlgkPQPaACBiMzk5ZWRjZTAyOGI0MTBkOGRkNTRjYzNkN2Q1NTFjOA> ，密码：p5Sb
  * Windows版本：<https://space.dingtalk.com/s/gwHOABmLzwLOGlgkPQPaACBhNzNjYjI5NTYxMzQ0NmUwOWRmMTFlN2UzMTYxNDQ4Mw> ，密码：V7ms
* 下载开发环境的虚拟机备份`ubuntu.vdi`（钉群群文件下载）。
* 打开VirtualBox，用刚刚下载的已有虚拟硬盘文件创建新的Ubuntu虚拟机，内存选择1024M。
* 启动新建实例，等待进入到登录阶段。用户名为`pouch`，密码为`123456`。

## 虚拟机安装PouchContainer

在Ubuntu上安装PouchContainer需要Ubuntu版本为16.04。其他的版本目前还未被测试和支持。PouchContainer与Docker冲突，所以在安装PouchContainer之前，你需要移除本机的Docker。

* 移除本机Docker。

  ```bash
  sudo apt-get remove docker
  ```

* 安装LXCFS，保证PouchContainer的强隔离性。LXCFS在Ubuntu上被默认安装。

  ```bash
  sudo apt-get install lxcfs
  ```

* 安装包以允许`apt`通过`HTTPS`来使用一个repository。

  ```bash
  sudo apt-get install curl apt-transport-https ca-certificates software-properties-common
  ```

* 添加PouchContainer官方GPG密钥。

  ```bash
  curl -fsSL http://mirrors.aliyun.com/opsx/pouch/linux/debian/opsx@service.alibaba.com.gpg.key | sudo apt-key add -
  ```

  通过搜索fingerprint的最后8个字符，确认你现在拥有带有fingerprint为`F443 EDD0 4A58 7E8B F645 9C40 CF68 F84A BE2F 475F`的密钥。

  ```bash
  $ apt-key fingerprint BE2F475F
  pub   4096R/BE2F475F 2018-02-28
        Key fingerprint = F443 EDD0 4A58 7E8B F645  9C40 CF68 F84A BE2F 475F
  uid                  opsx-admin <opsx@service.alibaba.com>
  ```

* 创建PouchContainer的repository。

  在你在一个全新的host machine上第一次安装PouchContainer之前，你要首先创建一个PouchContainer的repository。默认为`stable ` repository。如果要添加`test` repository，在以下命令的`stable`之后加上`test`即可。在此之后，你可以从这个repository里安装和更新PouchContainer。

  ```bash
  sudo add-apt-repository "deb http://mirrors.aliyun.com/opsx/pouch/linux/debian/ pouch stable"
  ```

* 安装最新版本的PouchContainer。

  ```bash
  # update the apt package index
  sudo apt-get update
  sudo apt-get install pouch
  ```

  安装PouchContainer之后，`pouch`组被创建，但组里没有用户。

## 虚拟机启动PouchContainer

* 启动PouchContainer。

  ```bash
  sudo service pouch start
  ```

  现在，你可以pull一个镜像并运行PouchContainer的容器。以`busybox`为例。

  * 启动一个busybox基础容器。

    ```bash
    sudo su
    pouch pull reg.docker.alibaba-inc.com/busybox:latest
    pouch run -t -d busybox sh
    ```

  * 登入启动的容器：

    ```bash
    pouch exec -it {ID} sh  # ID is the first 6 digits of the output of the last command
    ```

    此时，我们已经成功进入到这个container的终端。

* 关闭PouchContainer。

  ```bash
  sudo service pouch stop
  ```

## 配置共享文件夹（推荐）

### 推荐原因

由于在VirtualBox中的操作体验较差，不利于习惯了使用IDE或者Text Editor等工具编写调试代码的同学的开发，推荐利用VirtualBox的共享文件夹功能将host machine上的pouch代码文件夹挂载在虚拟机中，即可直接在host machine上进行开发，并在虚拟机中测试代码。

### 配置步骤（以Mac为例）

* 切换为root用户：

  ```bash
  pouch@ubuntu:~$ sudo su
  [sudo] password for pouch: ******
  ```

* 安装VirtualBox的客户端工具包，工具包可帮助实现VirtualBox的共享文件夹功能。

  ```bash
  root@ubuntu:/home/pouch$ apt-get update
  root@ubuntu:/home/pouch$ apt-get install virtualbox-guest-utils
  ```

* 在Linux虚拟机上建立空文件夹作为镜像路径，并修改文件夹读写权限，保证所有用户对该文件夹都可读可写可执行。

  由于GOPATH等配置，应将host machine目录挂载到`/root/gopath/`之后的路径。

  ```bash
  root@ubuntu:/home/pouch$ cd /root/gopath/
  root@ubuntu:~/gopath$ mkdir shareMac
  root@ubuntu:~/gopath$ chmod 777 shareMac/
  ```

  若使用默认的挂载路径`/media/sf_<shared folder name>`，需在`/etc/profile`中额外将`/media`路径添加到GOPATH中（可将以下命令直接粘贴在`/etc/profile`文件最后）。

  ```bash
  GOPATH=$GOPATH;/media 
  ```

* 在VirtualBox里配置host machine端的共享文件夹路径。

  关闭虚拟机并打开VirtualBox，点击虚拟机的<设置>--<共享文件夹>，添加新的文件夹。文件夹路径选择本地的从GitHub上clone下来的pouch文件夹路径，文件夹名字可以叫`work`（***注意这里名字不能和pouch文件夹同名***），点击选择下面的“自动挂载”和“固定分配”，点击“确定”完成配置。

  ![image-20180730192513951](https://i.loli.net/2018/07/30/5b5f2cbce4f54.png)

* 将配置好的host machine路径中的代码挂载到虚拟机中。

  ```bash
  root@ubuntu:/home/pouch$ mount -t vboxsf work /root/gopath/shareMac/
  ```

  `work`为VirtualBox中设置的共享文件夹别名，`/root/gopath/shareMac`为在虚拟机中设置的镜像路径。此时在`/root/gopath/shareMac`里已经可以看见pouch的代码。共享文件夹配置完成，在本地修改文件，在虚拟机里文件同步被修改。

  * 注意：如果这里报错，重启一下虚拟机即可。

  * 可以用`df -l`命令查看，出现以下信息即表示挂载成功。

    ```bash
    Filesystem                  1K-blocks      Used Available Use% Mounted on
    work                        244277768 115328120 128949648  48% /root/gopath/shareMac
    ```

* 如果想取消挂载，可以用`umount`命令。

  ```bash
  umount /root/gopath/shareMac/
  ```

* 注意：每次虚拟机重新启动时，都要重新挂载一遍。也就是执行一次`mount -t vboxsf work /root/gopath/shareMac/`命令。

## 配置虚拟机端口转发（推荐）

### 推荐原因

由于在VirtualBox中的操作体验较差，但配置共享文件夹步骤较为复杂，且容易遇到问题。配置虚拟机端口转发可以允许开发者在host machine上利用`ssh`将本地修改好的代码copy到虚拟机上进行测试。

### 配置步骤（以Mac为例）

* 打开VirtualBox里虚拟机的<设置>--<网络>--<高级>，选择<端口转发>。新建一个TCP协议，宿主端口随意设置，客户端口设为22，点击“确定”。

  ![image-20180730194345483](https://i.loli.net/2018/07/30/5b5f2cbc4a0d3.png)

* 在本地的Terminal利用`ssh`连接到虚拟机。

  ```bash
  ssh -p 2222 pouch@127.0.0.1
  ```

* 在本地的Terminal利用`scp`向虚拟机传输文件。

  ```bash
  scp -P 2222 <filename_on_host> pouch@127.0.0.1:<path_on_vm>
  ```