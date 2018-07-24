# introduction
PouchContainer, a enterprise-class container platform, only supports *linux operating system*. Pouch Container needs virtual machine to run and test locally. This guide is to instruct readers to build  PouchContainer environment based on VirtualBox + Ubuntu16.04 on mac or windows.
# Steps
## 1. Install and configure VirtualBox
1. [Download](https://www.virtualbox.org/) installation package for VirtualBox

![image.png | left | 540x420](https://cdn.nlark.com/lark/0/2018/png/121971/1532348887868-0915ef45-e8e0-4fca-9c7a-2d2118ef2434.png "")

2. in the installation wizard:</span>

![image.png | left | 540x420](https://cdn.nlark.com/lark/0/2018/png/121971/1532398838775-7ec9e175-e042-4e73-b1fd-82a9c6fa29ca.png "")


3. Choose a way to install the features. Use the default value here. Click 'Next' twice directly:

![image.png | left | 540x420](https://cdn.nlark.com/lark/0/2018/png/121971/1532398852608-54979f75-1829-41a8-bcd2-875bb4754544.png "")
![image.png | left | 540x420](https://cdn.nlark.com/lark/0/2018/png/121971/1532398858507-5bb842f2-1b08-4539-9130-913612f81356.png "")


4. There is a warning . Click 'YES'.

![image.png | left | 540x420](https://cdn.nlark.com/lark/0/2018/png/121971/1532398865097-1d1527b0-2c5e-4555-82d5-3e2297f1f352.png "")


5. Click 'Install'

![image.png | left | 540x420](https://cdn.nlark.com/lark/0/2018/png/121971/1532398879404-f14adf42-5e0f-41fd-8cad-ec9bbaceeb94.png "")


6. After installation, click 'Finish'

![image.png | left | 538x422](https://cdn.nlark.com/lark/0/2018/png/121971/1532398888522-6fd8eb46-fd76-4660-9ad2-e45b410c3657.png "")


## 2. Configure VirtualBox
1. Click 'New' in the VirtualBox Manager to create a new virtual machine

![image.png | left | 536x107](https://cdn.nlark.com/lark/0/2018/png/121971/1532399062624-5d4f4135-24a7-486b-b689-78e1833ad308.png "")


2. Set the virtual machine type and version

![image.png | left | 500x524](https://cdn.nlark.com/lark/0/2018/png/121971/1532398938160-e5097682-28d7-4c8d-907d-6bd3ba7b3352.png "")


3. Set the memory size. We use the default value here. Then click 'Next'

![image.png | left | 498x523](https://cdn.nlark.com/lark/0/2018/png/121971/1532398944630-cffd298a-4306-475e-8e0f-5b7d76d1d316.png "")


3. Choose how to create a new virtual hard disk. Here we choose 'Use the existing virtual hard disk file'. The Ubuntu has installed PouchContainer. If you don't have such a file, you can click 'Create virtual hard disk now', and you may need to configure the system and [install PouchContainer](https://github.com/alibaba/pouch/blob/master/INSTALLATION.md) manually:

![image.png | left | 502x522](https://cdn.nlark.com/lark/0/2018/png/121971/1532398973010-a70ea6d4-4be8-48ef-adea-4ffe542a341e.png "")


## 3. Enter the virtual machine, start and log in PouchContainer
1. After creating the virtual machine successfully, you can start the virtual machine, log in, and switch to the root user:</span></span>
```bash
sudo -i
```
2. check the network
```bash
ping www.alibaba-inc.com
```
3. Start the pouch service
```bash
systemctl start pouch
```
4. Start an existing busybox base container
```bash
pouch run -t -d busybox sh
```
5. If you don't have a busybox, you can execute the command like below
```bash
pouch pull busybox
```
6. Login in the started container, where the ID is the first six digits in the full ID from the output of startup command:
```bash
pouch exec -it {ID} sh
```

The result is shown as below:

![image.png | left | 747x113](https://cdn.nlark.com/lark/0/2018/png/121971/1532352167171-3167c9b6-9b49-4698-b056-e26073e64312.png "")

Now you have completed the environment building. Enjoy PouchContainer!
