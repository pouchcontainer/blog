# Enviroment Setup

## Main Links

- [Install Golang](#install-golang)
- [Install VS Code](#install-vs-code)
- [PouchContainer Setup](#pouchcontainer-setup)
  - [VirtualBox Downloads](#virtualbox-downloads)
  - [DingPan link](#dingPan-link)

## Install Golang

- Create Directories

  `mkdir $HOME/Go`
  
  `mkdir $HOME/Go`
  
  `mkdir -p $HOME/Go/src/github.com/user`

 

- Setup your paths

  `export GOPATH=$HOME/Go`
  
  `export GOROOT=/usr/local/opt/go/libexec`
  
  `export PATH=$PATH:$GOPATH/bin`
  
  `export PATH=$PATH:$GOROOT/bin`

 

- Install Go

  `brew install go`

 

- "go get" the basics

  `go get golang.org/x/tools/cmd/godoc`

 

## Install VS Code

VS Code Download: <https://code.visualstudio.com/download>



## PouchContainer Setup

PouchContainer is an enterprise-class container solution, it only supports the Linux operating system. Users of other operating systems need to use virtual machines to run and test locally. Please follow the steps below to set up the environment.

 
### VirtualBox Downloads
  <https://www.virtualbox.org/wiki/Downloads>   
  MacOS,  Windows users please click the corresponding link in the above url.
  
  DingPan link
  - Mac
      <https://space.dingtalk.com/s/gwHOABma4QLOGlgkPQPaACBiMzk5ZWRjZTAyOGI0MTBkOGRkNTRjYzNkN2Q1NTFjOA>   
      password: p5Sb
  - Windows
      <https://space.dingtalk.com/s/gwHOABmLzwLOGlgkPQPaACBhNzNjYjI5NTYxMzQ0NmUwOWRmMTFlN2UzMTYxNDQ4Mw>   
      password: V7ms
 
  Download the virtual machine backup of the development environment
  https://space.dingtalk.com/s/gwHOABmslALOGlgkPQPaACAwNTg4YTBjOGI4OTI0MGQ5YjE5MDgyYWFjMzAxMDY1MQ   
  paddword: tkD3 

  Open VirtualBox: Create a new instance, give whatever name you want to the instance. Choose system type as Linux. Choose version as Red Hat(64-bit).

  <img src="https://img.alicdn.com/tfs/TB1zfYLDv1TBuNjy0FjXXajyXXa-939-687.png" alt="Picture1" width="70%" >
 

  click continue to the next step. Set memory size as 1024M.

  <img src="https://img.alicdn.com/tfs/TB153GfDpOWBuNjy0FiXXXFxVXa-939-558.png" alt="Picture2" width="70%" >
 

  Click continue. Use the virtual disk file download in the last step. The CentOS.vdi file. Click “Create” to create a new instance. Start the new instance, login with users: root, password: Ali88Baiji

  <img src="https://img.alicdn.com/tfs/TB1afqRDuySBuNjy1zdXXXPxFXa-939-552.png" alt="Picture3" width="70%" >


  Use ip ad command to see the MAC address of your machine.

  `$ Ip ad`

  <img src="https://img.alicdn.com/tfs/TB1NVGpDER1BeNjy0FmXXb0wVXa-939-397.png" alt="Picture4" width="70%" >

  Use vim to modify /etc/sysconfig/network-scripts/ifcfg-eth0 file, modify the HWADDR same as the MAC address showed above. Save and exit.

  <img src="https://img.alicdn.com/tfs/TB1u_JCvnXYBeNkHFrdXXciuVXa-939-639.png" alt="Picture5" width="70%" >


  Reboot and ping [www.alibaba-inc.com](http://www.alibaba-inc.com) to see if the network connection is correct.

  <img src="https://img.alicdn.com/tfs/TB1CW8oviOYBuNjSsD4XXbSkFXa-939-216.png" alt="Picture6" width="70%" >

 
  `$ systemctl  start pouch`

  Start the pouch service.

  `$ pouch run -t -d busybox sh`

  Start a busybox basic container.

  <img src="https://img.alicdn.com/tfs/TB1jFuFDACWBuNjy0FaXXXUlXXa-939-87.png" alt="Picture7" width="70%" >

  `$ pouch exec -it {ID} sh`

  The ID is the first six characters showed in the last command.

  <img src="https://img.alicdn.com/tfs/TB1_G6bDxGYBuNjy0FnXXX5lpXa-939-75.png" alt="Picture8" width="70%" >

  Now, you have successfully start the PouchContainer service.
