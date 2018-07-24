# 1. Introduction to PouchContainer
 [PouchContainer](https://github.com/alibaba/pouch) is an efficient, lightweight, enterprise-class rich container engine technology that is open sourced by Alibaba Group. It is characterized by strong isolation and portability. PouchContainer can be used to help enterprises quickly realize the containerization of inventory business and improve the utilization of physical resources inside the enterprise.

The Pouch Container originated from the internal scene of Alibaba. At the beginning of its birth, Alibaba engineers put lots of efforts to secure the Internet application. Strong isolation and portability are the best proof of it. 

Under Alibaba's mass scale, PouchContainer's support for the business has been tested by “11.11” in an unprecedented manner. After being open sourced, Pouch Containers became an inclusive technology, positioning itself to “help the enterprise to quickly realize the containerization of the stock business”.
 # 2. Experience to Environmental Building
 【Note】The session 2 is just a guidance for a green hand, the pouch dictionary in this image clones the repo in alibaba/pouch directly, you can not submit in this dictionary!
 ## 2.1. Installing VirtualBox
 1. [AliLang]->[Manager]->[Software download], see follows:
 ![undefined](https://cdn.nlark.com/lark/0/2018/png/132231/1532344205392-0ba64dae-b407-4ff7-b8b9-f221a93ae728.png)
 2. Inputting "VirtualBox" in search box, and install:
 ![undefined](https://cdn.nlark.com/lark/0/2018/png/132231/1532344290135-ed2e9b67-766e-45ef-a72a-9ff034f27e22.png) 
You can download on the [DingPan] if you don't have [AliLang]:
Mac version:
https://space.dingtalk.com/s/gwHOABma4QLOGlgkPQPaACBiMzk5ZWRjZTAyOGI0MTBkOGRkNTRjYzNkN2Q1NTFjOA  password: p5Sb
Windows version:
https://space.dingtalk.com/s/gwHOABmLzwLOGlgkPQPaACBhNzNjYjI5NTYxMzQ0NmUwOWRmMTFlN2UzMTYxNDQ4Mw  password: V7ms
## 2.2. Downloading Virtual Machine Backup
Downloading `ubuntuPouch.vdi` in the chat group, this file costs 3.5GB.
## 2.3. Installing Virtual Machine Environment
1. Choosing "new" in Oracle VM VirtualBox->Choosing a name whatever you want->'Linux' type->'Ubuntu (64-bit)' version->Next.
2. Use 1024M memory usually:<br>
![undefined](https://cdn.nlark.com/lark/0/2018/png/132231/1532345103260-6995b132-bbdc-49e5-93fd-cfbe30fb3f79.png)
3. [Existent virtual hard disk file]->choosing `.vdi` in the session 2.2, then 'CREAT'
![undefined](https://cdn.nlark.com/lark/0/2018/png/132231/1532345138776-73add3b0-9f87-45e0-bf4b-9db6ef7d9b06.png) 
## 2.4. Starting New Case
1. login name: pouch, password:123456
![undefined](https://cdn.nlark.com/lark/0/2018/png/132231/1532345382824-733e22eb-234c-4c8f-b22c-f1cd2a7a8553.png) 
2. Inputting `sudo su` to change the user:
![undefined](https://cdn.nlark.com/lark/0/2018/png/132231/1532345631234-fd78bbb1-fafa-4bc5-b972-b688b56eaec6.png) 
## 2.5. Checking the network
Inputting `ping alibaba-inc.com` to check the status of network.
## 2.6. Starting Pouch service
1. Inputting `systemctl start pouch`
2. Inputting `pouch run -t -d busybox sh` to start a busybox container
3. Inputting `pouch exec -it {ID} sh` to login. {ID} is the first six number in the last command
![undefined](https://cdn.nlark.com/lark/0/2018/png/132231/1532346183142-78265ff4-2b39-4947-9221-5bc2642bdd05.png) 
Till now, you can run PouchContainer in the environment of VirtualBox + Ubuntu16.04. This environment include the tools like vim, make, git, go and so on. The source code of pouch is in the path of `/root/gopath/src/github.com/alibaba/pouch`
# 3. Mounting shared folder
## 3.1. Downloading the latest pouch code
1. Configure git, such as user name or email
2. Inputting `git clone@github.com:Qiaoxinshu/pouch.git` in git
![undefined](https://cdn.nlark.com/lark/0/2018/png/132231/1532347763099-2c1368c2-ccce-436b-898e-e2f965d0c4e2.png) 
## 3.2. Adding shared folder
[Settings]->[Shared folder]->[Add], choosing [fixed allocation]
![undefined](https://cdn.nlark.com/lark/0/2018/png/132231/1532347966375-ae70577f-a34c-41da-afda-b5cf8636da95.png) 
then we will see the result:
![undefined](https://cdn.nlark.com/lark/0/2018/png/132231/1532348064973-0deaacc6-5e49-4c27-83f7-e2410d4afcef.png)
## 3.3. Mounting Virtual Machine
1. choosing path: `/root/gopath/src/github.com/alibaba/`
2. remove the original pouch file: `sudo rm -rf pouch`
3. Downloading VitualBox enhanced function: `sudo apt-get install virtualbox-guest-x11`, Note that you should restart the virtual machine.
4. Mounting pouch dictionary: `sudo mount -t vboxsf pouch/ pouch/`.
![undefined](https://cdn.nlark.com/lark/0/2018/png/132231/1532349559864-9fa068fa-05b7-4231-9f36-6dfca1719a5c.png) 
