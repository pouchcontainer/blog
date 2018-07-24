# introduction
PouchContainer, a enterprise-class container platform, only supports *linux operating system*. Pouch Container needs virtual machine to run and test locally. This guide is to instruct readers to build  PouchContainer environment based on VirtualBox + Ubuntu16.04 on mac or windows.
# Steps
## 1. Install and configure VirtualBox
1. [Download](https://www.virtualbox.org/) installation package for VirtualBox
![image.png | left | 540x420](https://cdn.nlark.com/lark/0/2018/png/121971/1532348887868-0915ef45-e8e0-4fca-9c7a-2d2118ef2434.png "")

2. in the installation wizard:</span>
![image.png | left | 540x420](https://cdn.nlark.com/lark/0/2018/png/121971/1532349130108-3b9c506d-f652-4add-9914-8c8ea7e926ec.png "")


3. Choose a way to install the features. Use the default value here. Click 'Next' twice directly:
![image.png | left | 540x420](https://cdn.nlark.com/lark/0/2018/png/121971/1532349296176-4b100caa-b402-4f21-aecc-8aff21a4f427.png "")
![image.png | left | 540x420](https://cdn.nlark.com/lark/0/2018/png/121971/1532349417293-6afb7e2b-713f-45ae-bbca-0258af8cc2c9.png "")


4. There is a warning . Click 'YES'.
![image.png | left | 540x420](https://cdn.nlark.com/lark/0/2018/png/121971/1532349540089-9040e338-406e-401c-8792-5963b557b688.png "")


5. Click 'Install'
![image.png | left | 540x420](https://cdn.nlark.com/lark/0/2018/png/121971/1532349617023-ccf1d26e-3aa5-431e-9764-21545b5359bc.png "")


6. After installation, click 'Finish'
![image.png | left | 538x422](https://cdn.nlark.com/lark/0/2018/png/121971/1532349733085-bcbf691a-a022-4eaa-96dd-a73c9f02a0b1.png "")


## 2. Configure VirtualBox
1. Click 'New' in the VirtualBox Manager to create a new virtual machine
![image.png | left | 536x107](https://cdn.nlark.com/lark/0/2018/png/121971/1532350044579-5118ca49-cc95-4d35-b97f-37064c7b2898.png "")


2. Set the virtual machine type and version
![image.png | left | 500x524](https://cdn.nlark.com/lark/0/2018/png/121971/1532350116607-08386785-f566-47de-a9da-636ad733c2d0.png "")


3. Set the memory size. We use the default value here. Then click 'Next'
![image.png | left | 498x523](https://cdn.nlark.com/lark/0/2018/png/121971/1532350251616-d563bc78-41ae-4dec-a55a-861134fd6a31.png "")


3. Choose how to create a new virtual hard disk. Here we choose 'Use the existing virtual hard disk file'. The Ubuntu has installed PouchContainer. If you don't have such a file, you can click 'Create virtual hard disk now', and you may need to configure the system and [install PouchContainer](https://github.com/alibaba/pouch/blob/master/INSTALLATION.md) manually:
![image.png | left | 502x522](https://cdn.nlark.com/lark/0/2018/png/121971/1532350669919-c7f99d3a-fb3e-4a69-a792-4f16c3657e4c.png "")


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
