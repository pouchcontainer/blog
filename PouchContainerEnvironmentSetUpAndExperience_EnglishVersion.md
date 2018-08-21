# PouchContainer Environment Set-Up and Experience

The build and experiences of running PouchContainer environment is based on `macOS High Sierra 10.13.6` as the host and `VirtualBox + Ubuntu16.04 ` as the virtual machine.

### 1. INSTALL VIRTUAL MACHINE

The author's host system is macOS, use Homebrew to install VirtualBox, please refer to Homebrew:
[https://brew.sh/](https://brew.sh/)

```shell
brew cask search virtualbox
```

Run virtualBox
![undefined](https://cdn.nlark.com/lark/0/2018/png/117932/1532345813010-47020fd7-5bd3-4e0c-8342-45a8bb7ea9df.png) 

Create a new virtual machine
![undefined](https://cdn.nlark.com/lark/0/2018/png/117932/1532345846188-a4c14f90-f965-434a-8e78-b006c79fa75f.png) 

Select the ubuntuPouch.vdi from DingDing group
![undefined](https://cdn.nlark.com/lark/0/2018/png/117932/1532345864538-dd950de1-f859-4f7f-b508-8ed90c7813a6.png) 

### 2. RUN VIRTUAL MACHINE

Set up port forwarding in the virtual machine so that the host machine can create ssh conection to the virtual machine
![undefined](https://cdn.nlark.com/lark/0/2018/png/117932/1532345957338-86584231-8425-4c05-816e-11e3a7ec83b5.png) 
![undefined](https://cdn.nlark.com/lark/0/2018/png/117932/1532345971592-5cd59593-d240-4e22-95e9-f8347f976bc4.png) 

Run the virtual machine. The account is pouch, and the password 123456. Remind to check if the virtual machine instance opens port 22. If it is, then exit instance
and enter the following command line in the host。
```shell
ssh -l pouch -p 1111 127.0.0.1
```
![undefined](https://cdn.nlark.com/lark/0/2018/png/117932/1532346199103-783b7671-9b6a-490c-8ad8-792774381882.png) 
Switch to root 
```shell
sudo su root
```

### 3. RUN POUCH

Create pouch instance

```shell

pouch run -t -d busybox sh

```

Maintain the return value for future usage

```shell

pouch exec -it <key> sh

```
Enter into the pouch container
![undefined](https://cdn.nlark.com/lark/0/2018/png/117932/1532346347168-617f5ffe-5e1f-4519-96ae-28fccd05fbf5.png) 