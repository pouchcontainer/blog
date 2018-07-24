# PouchContainer Environment Setup Tutorial (PouchContainer + Ubuntu16.04 + VirtualBox)

## Download VirtualBox

#### 1. Download VirtualBox from ALILANG-Manager-Software Download, default version is 5.2.12
#### 2. Download VirtualBox from DingPan

- Mac Version : [Mac Version VirtualBox Download](https://space.dingtalk.com/s/gwHOABma4QLOGlgkPQPaACBiMzk5ZWRjZTAyOGI0MTBkOGRkNTRjYzNkN2Q1NTFjOA)   Password：p5Sb
- Windows Version :[Windows Version VirtualBox Download](https://space.dingtalk.com/s/gwHOABmLzwLOGlgkPQPaACBhNzNjYjI5NTYxMzQ0NmUwOWRmMTFlN2UzMTYxNDQ4Mw)   Password：V7ms

## Configuring Virtual Machine

#### 1. Download the virtual machine backup of the development environment
- From DingDing Group's files(ubuntuPouch.vdi)

#### 2. Open VirtualBox
- Open VirtualBox, Click on New->Customize VM name->Choose the OS type【Linux】and version【Ubuntu (64-bit)】-> Click on Next

![EBE73F08-C433-4c29-88AE-855AF42D7855.png](https://cdn.nlark.com/lark/0/2018/png/117855/1532344347704-e48d21d8-50f7-4f2c-aae9-21381f5e9c5e.png) 

- Choose 【1024M】-> Click on Next

![E53144FD-CF7C-4275-992B-7B668EB9D70A.png](https://cdn.nlark.com/lark/0/2018/png/117855/1532345740535-34e7d297-fc7d-4f32-a1b4-7a583e6ea5d3.png) 

- Choose 【Use an existing virtual hard disk file】->Choose ubuntuPouch.vdi-> Click on Create

![F73BF283-5883-4bf0-8950-CA825A897C54.png](https://cdn.nlark.com/lark/0/2018/png/117855/1532346451698-be17021d-977f-4834-80c3-e485460e867a.png) 

#### 3. Start VM, login using username 'pouch' and password '123456'
![2F3071DE-4624-4de5-A242-4B8277AAFDD3.png](https://cdn.nlark.com/lark/0/2018/png/117855/1532346205714-cc98c909-6371-48e6-acf1-614001d71df4.png) 

#### 4. Login as root
- exec 'sudo -i'
- input the password

![EA9BA643-9DA5-4503-B530-9107C0A2B786.png](https://cdn.nlark.com/lark/0/2018/png/117855/1532346621286-a288a971-a72d-4c71-8a49-6953f7c3e27f.png) 

#### 5. Check the network 
- Try the command 'ping www.alibaba.com' to check the network

![0B820657-710B-4234-980D-73EAFA134D0E.png](https://cdn.nlark.com/lark/0/2018/png/117855/1532346740950-3939b8e4-2545-4a77-bbaf-b47f266a388a.png) 

## Run PouchContainer

#### 1. Start pouch service
- exec 'systemctl start pouch' to start pouch service(Auto-start by default)

#### 2. Start busybox container
- exec 'pouch run -t -d busybox sh' to start a busybox container, the output is container ID

#### 3. Login to container
- exec 'pouch exec -it {ID} sh' to login to the container, ID is the first six digits of the complete ID that the previous command output

![D7504959-8225-4fdf-847C-BE447782DAE9.png](https://cdn.nlark.com/lark/0/2018/png/117855/1532348031330-c0f59b6a-a127-4d8e-ad04-7ebd564a91e4.png) 
