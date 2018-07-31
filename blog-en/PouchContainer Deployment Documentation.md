## PouchContainer Introduction

PouchContainer is an open-source project created by Alibaba Group to promote the container technology movement. It provides applications with a lightweight runtime environment with strong isolation, high portability and minimal overhead, which can help enterprises to quickly achieve business containerization, and improve the resource utilization ratio in large scale data center at the same time.

## GitHub User Configuration and Source Code Download

- GitHub user configuration.

  ```bash
  git config --global user.name "<your_github_username>"  # set Github username
  git config --global user.email "<your_github_email>"  # set Github email
  git config --global credential.helper store  # avoid typing password every time during pulling
  ```

- Open pouch source code repository in your browser: <https://github.com/alibaba/pouch>.

- Click "Fork" on the upper right corner.

- Clone pouch repository in the local terminal.

  ```bash
  git clone https://github.com/<your_github_username>/pouch.git
  ```

## Create and Start a Ubuntu Virtual Machine

- Install VirtualBox. Use Alilang--Manager--Software Download to install, the default version is 5.2.12. Links to DingPan are listed below:
  - Mac Version: <https://space.dingtalk.com/s/gwHOABma4QLOGlgkPQPaACBiMzk5ZWRjZTAyOGI0MTBkOGRkNTRjYzNkN2Q1NTFjOA>. Password: p5Sb
  - Windows Version: <https://space.dingtalk.com/s/gwHOABmLzwLOGlgkPQPaACBhNzNjYjI5NTYxMzQ0NmUwOWRmMTFlN2UzMTYxNDQ4Mw>. Password: V7ms
- Download the `.vdi` file for VM from Group Chat in Dingding.
- Open VirtualBox, create a new Ubuntu Virtual Machine by the `.vdi` file you just downloaded. Choose 1024M for memory.
- Start the new VM, wait to login. Username is `pouch`, password is `123456`.

## Install PouchContainer in VM

To install PouchContainer, you need a maintained version of Ubuntu 16.04 (Xenial LTS). Archived versions aren't supported or tested.

PouchContainer is conflict with Docker, so you must uninstall Docker before installing PouchContainer.

- Remove Docker.

  ```bash
  sudo apt-get remove docker
  ```

- Install LXCFS. PouchContainer supports LXCFS to provide strong isolation. By default, LXCFS is enabled.

  ```bash
  sudo apt-get install lxcfs
  ```

- Install packages to allow `apt` to use a repository over `HTTPS`.

  ```bash
  sudo apt-get install curl apt-transport-https ca-certificates software-properties-common
  ```

- Add PouchContainer's official GPG key.

  ```bash
  curl -fsSL http://mirrors.aliyun.com/opsx/pouch/linux/debian/opsx@service.alibaba.com.gpg.key | sudo apt-key add -
  ```

  Verify that you now have the key with the fingerprint `F443 EDD0 4A58 7E8B F645 9C40 CF68 F84A BE2F 475F`, by searching for the last 8 characters of the fingerprint.

  ```bash
  $ apt-key fingerprint BE2F475F
  pub   4096R/BE2F475F 2018-02-28
        Key fingerprint = F443 EDD0 4A58 7E8B F645  9C40 CF68 F84A BE2F 475F
  uid                  opsx-admin <opsx@service.alibaba.com>
  ```

- Set up the PouchContainer repository

  Before you install PouchContainer for the first time on a new host machine, you need to set up the PouchContainer repository. We enabled `stabel` repository by default, you always need the `stable` repository. To add the `test`repository, add the word `test` after the word `stable` in the command line below. Afterward, you can install and update PouchContainer from the repository.

  ```bash
  sudo add-apt-repository "deb http://mirrors.aliyun.com/opsx/pouch/linux/debian/ pouch stable"
  ```

- Install the latest version of PouchContainer.

  ```bash
  # update the apt package index
  sudo apt-get update
  sudo apt-get install pouch
  ```

  After installing PouchContainer, the `pouch` group is created, but no users are added to the group.

## Start PouchContainer in VM

- Start PouchContainer。

  ```bash
  sudo service pouch start
  ```

  Afterwards, you can pull an image and run PouchContainer containers. Take `busybox` for example.

  - Start a busybox container。

    ```bash
    sudo su
    pouch pull reg.docker.alibaba-inc.com/busybox:latest
    pouch run -t -d busybox sh
    ```

  - Login to the container：

    ```bash
    pouch exec -it {ID} sh  # ID is the first 6 digits of the output of the last command
    ```

    Right now, we have successfully access to the terminal of container.

- Stop PouchContainer。

  ```bash
  sudo service pouch stop
  ```

## Configure Shared Folders (Recommended)

### Reasons for Recommendation

Due to the poor operation experience in VirtualBox, some people who are accustomed to developing by an IDE or Text Editor may find it hard to test and debug on VirtualBox. We recommend you to configure shared folders in VirtualBox to mount the files on your host machine to your VM, which enables developing on your host machine, and testing on your VM without extra operations.

### Configuration Steps (On Mac)

- Switch to root user:

  ```bash
  pouch@ubuntu:~$ sudo su
  [sudo] password for pouch: ******
  ```

- Install VirtualBox Guest Utilities, which can help VirtualBox to enable Shared Folders.

  ```bash
  root@ubuntu:/home/pouch$ apt-get update
  root@ubuntu:/home/pouch$ apt-get install virtualbox-guest-utils
  ```

- Create a new empty folder on your VM, and modify its access permissions to enable all users to read, write and execute.

  Due to the configuration of GOPATH, you should mount the shared folder to the path behind `/root/gopath` on VM.

  ```bash
  root@ubuntu:/home/pouch$ cd /root/gopath/
  root@ubuntu:~/gopath$ mkdir shareMac
  root@ubuntu:~/gopath$ chmod 777 shareMac/
  ```

  If you want to use the default mounting path, i.e. `/media/sf_<shared folder name>`, you need to add `/media` into GOPATH either. (You can just copy and paste the below code as the last line of `/etc/profile`)

  ```bash
  GOPATH=$GOPATH;/media 
  ```

- Configure the shared folder in your host machine.

  Close your VM and open VirtualBox. Click Settings--Shared Folders--Add Shared Folders of this VM. Choose the pouch path cloned from GitHub as the `Folder Path`, set "word" as the `Folder Name` (MENTION: `Folder Name` cannot be the same name as the pouch folder on your host). Click "Auto Mount" and "Make Permanent", and click "OK" to finish.

  ![image-20180730192513951](https://i.loli.net/2018/07/30/5b5f2cbce4f54.png)

- Mount the shared folder on your VM.

  ```bash
  root@ubuntu:/home/pouch$ mount -t vboxsf work /root/gopath/shareMac/
  ```

  `work` is the `Folder Name` set in VirtualBox, `/root/gopath/shareMac` is the shared folder path you set in VM. Currently, you can see the pouch code in `/root/gopath/shareMac`. Configuration is done, files in your VM will be changed at the same time when you modify files in your host machine.

  - MENTION: if errors occur, just reboot the VM.

  - Check by `df -l` command. Mounting is successful if the below information is shown.

    ```bash
    Filesystem                  1K-blocks      Used Available Use% Mounted on
    work                        244277768 115328120 128949648  48% /root/gopath/shareMac
    ```

- Use `umount` command to delete mounting.

  ```bash
  umount /root/gopath/shareMac/
  ```

- MENTION: you should re-mount again whenever your VM is rebooted, i.e. run `mount -t vboxsf work /root/gopath/shareMac/`.

## Configure Port Forwarding (Recommended)

### Reasons for Recommendation

Due to the poor operation experience in VirtualBox as well as the complicated steps to configure shared folders, port forwarding becomes another recommended development technique. It enables developers to `ssh` to the VM and copy modified files to the VM.

### Configuration Steps (On Mac)

- Open Settings--Network-Advanced--Port Forwarding. Create a new TCP protocol, set the `Host Port` as you want, and set the `Guest Port` as 22. Click "OK".

  ![image-20180730194345483](https://i.loli.net/2018/07/30/5b5f2cbc4a0d3.png)

- Connect to your VM via `ssh` in Terminal.

  ```bash
  ssh -p 2222 pouch@127.0.0.1
  ```

- Copy files to your VM via `scp` in Terminal.

  ```bash
  scp -P 2222 <filename_on_host> pouch@127.0.0.1:<path_on_vm>
  ```