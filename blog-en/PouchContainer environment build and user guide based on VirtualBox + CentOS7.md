#PouchContainer environment build and user guide based on VirtualBox + CentOS7

## Download VirtualBox

**1. ALILANG-Manager-Software Download-Download VirtualBox，default version is 5.2.12**

**2. Download VirtualBox from DingPan**

- Mac  Version Address: https://space.dingtalk.com/s/gwHOABma4QLOGlgkPQPaACBiMzk5ZWRjZTAyO GI0MTBkOGRkNTRjYzNkN2Q1NTFjOA  Password: p5Sb 
- Windows Version Address: https://space.dingtalk.com/s/gwHOABmLzwLOGlgkPQPaACBhNzNjYjI5NTYxM zQ0NmUwOWRmMTFlN2UzMTYxNDQ4Mw Password: V7ms 

## Configuring Virtual Machine

**1. Download the virtual machine backup of the development environment，DingPan Address: https://space.dingtalk.com/s/gwHOABmLxALOGlgkPQPaACA4N2JjNmIwMGI5N WU0MGE1YjZhNTBiOGNjMDZhOTJiNQ Password: pLgb**

**2. Open VirtualBox**

- New-Custom Name-Type choose【Linux】-Version choose【Red Hat(64- bit)】 

<img src="https://cdn.yuque.com/lark/0/2018/png/72820/1531142367376-58af5f86-e7e1-452e-aee3-cce1489a1a92.png" width="450px" height="300px" />

- RAM choose【1024M】

<img src="https://cdn.yuque.com/lark/0/2018/png/72820/1531142356237-1525281e-6fbd-4dfa-b359-8f10f9720ef6.png" width="450px" height="300px" />

- choose 【Existing virtual hard disk file】-choose vdi file downloaded from step 1-create

<img src="https://cdn.yuque.com/lark/0/2018/png/72820/1531142341940-d29a6743-c8e5-4581-adda-4e4f1149f1d2.png" width="450px" height="300px" />

**3. Start new instanse，waiting to enter the login stage，Username: `root`，Password:`Ali88Baiji `**

<img src="https://cdn.yuque.com/lark/0/2018/png/72820/1531142322664-3227342d-fa41-48b3-834e-3aa46b8446a2.png" width="700px" height="150px" />

**4. modify `/etc/sysconfig/network-scripts/ifcfg-eth0`，make the HWADDR match the MAC address displayed across the ip ad command，reboot , then  ping www.alibaba-inc.com, check if the network is work**

- show MAC address across the ip ad command

<img src="https://cdn.yuque.com/lark/0/2018/png/72820/1531142281019-3a911e83-76fb-47e9-b11e-3d97a960fbd7.png" width="700px" height="400px" />

- modify HWADDR in` /sysconfig/network-scripts/ifcfg-eth0`

<img src="https://cdn.yuque.com/lark/0/2018/png/72820/1531142305340-0966e48f-2bdc-492a-8277-791a92c62e0f.png" width="700px" height="130px" />

- ping www.alibaba-inc.com , check if the network is work

<img src="https://cdn.yuque.com/lark/0/2018/png/72820/1531142294988-03c54bb0-6778-4fb1-b149-9f5fa3e1122e.png" width="700px" height="200px" />



## Run PouchContainer

1. exec `stemctl start pouch`，start pouch service 
2. exec `pouch run -t -d busybox sh`， start a  busybox basic container
3. exec `pouch exec -it {ID} sh` , login to the started container，the ID is the first six digits of the full ID of the previous command output

<img src="https://cdn.yuque.com/lark/0/2018/png/72820/1531142287443-e03b22c0-06d9-48ab-a13f-ea8b6718877d.png" width="700px" height="100px" />

## Config sshd

**1. Set the NIC in VirtualBox and choose the host-only adapter**

<img src="https://cdn.yuque.com/lark/0/2018/png/72820/1531142079098-7c0cc1d6-3d78-4194-8d44-5cbc6e7ff2bb.png" width="450px" height="250px" />

**2. Modify the sshd configuration in the virtual machine to allow remote connections**

<img src="https://cdn.yuque.com/lark/0/2018/png/72820/1531142261139-1571f8f5-cc5a-4610-963c-b4930a16b69c.png" width="550px" height="300px" />

<img src="https://cdn.yuque.com/lark/0/2018/png/72820/1531143160892-bf7a520c-fc96-40b6-a9bf-d4286c9673ca.png" width="550px" height="300px" />

**3. Restart the sshd service：`systemctl restart sshd.service `**

**4. View the virtual machine IP address：`ip addr `**

<img src="https://cdn.yuque.com/lark/0/2018/png/72820/1531142232272-1d4b4cb6-623a-4336-8b41-e8568458814d.png" width="450px" height="300px" />

**5. Connect virtual machines through iTerm on the host**

 <img src="https://cdn.yuque.com/lark/0/2018/png/72820/1531143146304-720f9ef3-1031-4b73-aa60-250c44c41c26.png" width="700px" height="150px" />

## Mount the git repo folder to the VM 

**1.  Set up a shared folder in VirtualBox**

<img src="https://cdn.yuque.com/lark/0/2018/png/72820/1531142079098-7c0cc1d6-3d78-4194-8d44-5cbc6e7ff2bb.png" width="500px" height="300px" />

**2. Execute the following commands in the VM to install the required modules for mounting**

```
yum clean all
yum update
yum install kernel 
yum install kernel-devel 
yum install kernel-headers 
yum install gcc 
yum install make   
reboot 
cd /opt/VBoxGuestAdditions-*/init  
./vboxadd setup 
reboot
```

**3. exec `mount -t vboxsf pouchShare /root/go/src/github.com/alibaba/pouch ` to mount the shared folder to the VM's source path of pouch**

​    



