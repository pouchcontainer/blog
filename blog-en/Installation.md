# Getting Started with PouchContainer
## Installing VirtualBox
- Open [VirtualBox Downloads](https://www.virtualbox.org/wiki/Downloads)
- Choose the download link that matches your platform, then download and install VirtualBox.

## Downloading Ubuntu/CentOS Image
- Download``UbuntuPouch.vdi`` or ``CentOS.vdi.zip`` from shared Files in DingTalk. If you pick the later one you'll need to unzip it as well. In either case you will have either``UbuntuPouch.vdi`` or ``CentOS.vdi``

## Creating new VirtualBox Image
- Open the brand new VirtualBox you've just installed ![vb](./vb.png), then click the ``New`` button at top left.  
- Give your virtual machine a name and type in in the ``Name`` section，for the ``Type`` section you should choose Linux. In ``Version``, if you are using the CentOS image you should pick``Other Linux(64Bit)``, otherwise if you are using the Ubuntu image then you should pick``Ubuntu(64-bit)``. As shown in the image:
![vb2](./vb2.png)
- Allocate memory for the virtual machine, it's recommended for you to reasonably allocate RAM and you should allocate at least 1024MB.
- Enter HardDisk Creation Menu, choose ``Use an existing virtual hard disk file``, click the circled button then select the ``.vdi``file you created in step 1. Here we use Ubuntu as an example![vb3](./vb3.png)
- You've finished creating VM, click ``Start`` to run the Virtual Machine![vbdone](./vbdone.png)

## Log into the Virtual Machine and use Pouch
After the Virtual Machine has booted, the interface looks like the following:
![login](./login.png)
Enter your Username and Password.
Ubuntu Username：pouch, Password: 123456
Centos Username:root, Password: Ali88Baiji
For the following command if you are using Ubuntu you'll need to add ``sudo `` prefixing the command then use the login password ``123456`` to authorize.
Execute command ``systemctl start pouch`` to start pouch service
Execute command ``pouch run -t -d busybox sh`` to start a basic busybox container, the output looks like the following:
![pouchrun](./pouchrun.png)
Execute command``pouch exec -it {ID} sh``to log into the container, ``ID`` here is the first six digits of the output of the previous command, in the case it's ``ab9ad6``, as shown in the image: ![pouchrun2](./pouchrun2.png)
