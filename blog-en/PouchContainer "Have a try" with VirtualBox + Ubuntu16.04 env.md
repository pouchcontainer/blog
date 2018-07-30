# PouchContainer "Have a try" with VirtualBox + Ubuntu16.04 env.

## Introduction
PouchContainer, an open-source `enterprise rich container technology` developed by `Alibaba`, target at high effectiveness and lightweight. It also has some great characteristics such as high isolation，strong portability and less resource usage.

Since PouchContainer is an enterprise solution, it is only supported on `Linux`. So the virtual machine is needed for execution and test on the local environment.

## The Installation of VirtualBox & The Import of Image

- VirtualBox Install Package can be downloaded at the [Official WebSite](https://www.virtualbox.org/wiki/Downloads) or through `Alilang`.
![](https://i.loli.net/2018/07/30/5b5ed5e30c753.png)

- Recommended Linux Distribution and Version
    - Ubuntu 16.04
    - CentOS 7

> In this passage, the Distribution and Version that used in the Image is Ubuntu 16.04 (64-bit)

### Steps for Importing Image

1. Open VirtualBox, `create` an instance, select `Linux` - `Ubuntu(64-bit)`, name can be customized
![](https://i.loli.net/2018/07/30/5b5ed82dc96e8.png)
> If you can only choose a version of 32-bit, you can try to turn on the `Virtualization` in the Host Machine's BIOS, and restart
2. Select `Continue`, select `1024M` for memory, and then continue
3. Select `Use Existing Virtual Hard Disk File`, then import the `Trail Version Image`, click `create`
![](https://i.loli.net/2018/07/30/5b5ed8d04536f.png)

### Steps for Starting Service
1. Start the instance that we just created, log in with `username: pouch / password: 123456`
![](https://i.loli.net/2018/07/30/5b5eda998850b.png)
2. Switch to root user
``` bash
sudo su
```
3. Check the network
``` bash
ping www.alibaba-inc.com
```
4. Start pouch service
``` bash
systemctl start pouch
```
5. Create a new basic busybox container, which will generate a key
``` bash
pouch run -t -d busybox sh
```
6. Sign into the started container
``` bash
pouch exec -it {ID}
```
> {ID} means top six characters of the key

After finishing these steps, a new basic container has already been started and successfully log in.
![](https://i.loli.net/2018/07/30/5b5edd8c3f50f.png)

### Development Environment Configuration
The tools included in the trial version image are vim, make, git, go and other basic tools. And the source code of pouch is located at /root/gopath/src/github.com/alibaba/pouch.

- For users who are familiar with Vim development, you can develop in the virtual machine (replace the files in the pouch directory with the project files they folk to your own repository).

- For users who are familiar with IDE development, you can pull the code to the host machine and then mount the directory to the source directory of the pouch in the virtual machine.

### GitHub Configuration
Use HTTPS or SSH to clone the remote code to local. 

SSH is recommended here, which can save the trouble of entering the password every time you submit the code to the remote end. But the SSH key needs to be added under your personal GitHub account first. Details of generating and adding SSH key can refer to the [github official document](https://help.github.com/articles/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent/)

On the host machine, you still need to configure your GitHub account as follows:

``` git
git config --global user.name YOUR_USERNAME
git config --global user.email YOUR_EMAIL
```
### Multiple Github Accounts Configuration on Host Machine
There may be a need to configure multiple GitHub accounts on the host machine, which can be resolved through the configuration file:

``` bash
# Generate a public key by ssh-keygen, for example, generate under ~/.ssh
ssh-keygen -t rsa -b 4096 -C "your.email@address.com"

# Add the public key file to the corresponding site

# Add the private key to the SSH agent (replace the file with the private key just generated)
ssh-add -K ~/.ssh/id_rsa 
```
After completing the steps above, create a config file under ~/.ssh and write it in the following format: 
- Host: can be customized
- HostName: the site address
- User: git
- IdentityFile: the private key address
``` bash
# first.github (first@gmail.com)
Host github.com
HostName github.com
User git
IdentityFile ~/.ssh/id_rsa

# second (second@gmail.com)
Host github-second
HostName github.com
User git
IdentityFile ~/.ssh/id_rsa_second
```

### File Mounting on Host Machine 
When mounting,  enhanced plug-ins for VirtualBox are needed, which can be installed as follows:
``` bash
apt install virtualbox-guest-x11
```
In VirtualBox，Set the project directory folder of the host machine to `shared folder`, and choose `automatic mount` and `fixed assignment`。
![](https://i.loli.net/2018/07/30/5b5eee8226510.png)
go to the directory /root/gopath/src/github.com/alibaba/, remove the files from the original pouch directory(retain the pouch folder)，execute command to mount in the directory. (where SHARE_FOLDER_NAME is the name of the shared folder)
``` bash
sudo mount -t vboxsf SHARE_FOLDER_NAME pouch/
```
The pouch service can be started to detect if the configuration is correct.