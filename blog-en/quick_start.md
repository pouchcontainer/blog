# A quick tutorial about PouchContainer for VirtualBox and CentOS 7

## Install VirtualBox and CentOS 7
The download url of VirtualBox is [https://www.virtualbox.org/wiki/Downloads](https://www.virtualbox.org/wiki/Downloads), download the version according to your system, the article is based on the macOS. Install the software according to the guide after the installation finished.

The download url of CentOS 7 is [https://www.centos.org/download/](https://www.centos.org/download/), the article uses the minimal version. Start the VirtualBox, and create a new virtual machine instance following the guide. In the article we use the Red Hat 64-bit version Linux VM. 

After that, select the new virtual machine instance, mount the downloaded CentOS 7 iso file to CD-ROM. Start the virtual machine instance, and install CentOS following the guide. 

## Build and start up PouchContainer

While starting the virtual machine, if can not connect to the network, use the following command:
```
ip addr
```
find out the name of NIC. In the article the VM uses enp0s3, and run the following command:
```
vi /etc/sysconfig/network-scripts/ifcfg-enp0s3
```
Modiify the option `ONBOOT=no` to `ONBOOT=yes`, save and quit. And then restart the network service:
```
service network restart
```
Next, we will build and start PouchContainer service:

1. __Install yum-utils__
    ```
    sudo yum install -y yum-utils
    ```

2. __Set up the PouchContainer repossitory__
    ```
    sudo yum-config-manager --add-repo http://mirrors.aliyun.com/opsx/opsx-centos7.repo
    sudo yum update
    ```

3. __Install PouchContainer__
    ```
    sudo yum install pouch
    ```

4. __Start PouchContainer__
    ```
    sudo systemctl start pouch
    ```

5. start up the container
    ```
    pouch run -t -d busybox sh
    ```

6. log in the container, and 73cb2d are the first six numbers of return value of the last command line 
    ```
    pouch exec -it 73cb2d sh
    ```

7. Interact with the container
    ```
    echo "hello world"
    ```
And we finish the procedure right now.