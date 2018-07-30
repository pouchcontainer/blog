# Install and run PouchContainer with Virtual Box

This tutorial will guide you install the open source container Pouch from Alibaba with Virtual Box. And run a Ubuntu image with PouchContainer.

### 1. Open Virtual Box, and then install the Ubuntu 16.04

Install Ubuntu 16.04 with Virtual Box. Lauch the image of Ubuntu.

### 2. Install PouchContainer

**Prerequisites**

PouchContainer supports LXCFS to provide strong isolation, so you should install LXCFS firstly. By default, LXCFS is enabled.

``` bash
sudo apt-get install lxcfs
```

Install packages to allow 'apt' to use a repository over HTTPS:

``` bash
sudo apt-get install curl apt-transport-https ca-certificates software-properties-common
```

**1. Add PouchContainer's official GPG key**

``` bash
curl -fsSL http://mirrors.aliyun.com/opsx/pouch/linux/debian/opsx@service.alibaba.com.gpg.key | sudo apt-key add -
```

Verify that you now have the key with the fingerprint `F443 EDD0 4A58 7E8B F645  9C40 CF68 F84A BE2F 475F`, by searching for the last 8 characters of the fingerprint.

``` bash
$ apt-key fingerprint BE2F475F
pub   4096R/BE2F475F 2018-02-28
      Key fingerprint = F443 EDD0 4A58 7E8B F645  9C40 CF68 F84A BE2F 475F
uid                  opsx-admin <opsx@service.alibaba.com>
```

**2. Set up the PouchContainer repository**

Before you install PouchContainer for the first time on a new host machine, you need to set up the PouchContainer repository. We enabled `stabel` repository by default, you always need the `stable` repository. To add the `test` repository, add the word `test` after the word `stable` in the command line below. Afterward, you can install and update PouchContainer from the repository.

``` bash
sudo add-apt-repository "deb http://mirrors.aliyun.com/opsx/pouch/linux/debian/ pouch stable"
```

**3. Install PouchContainer**

Install the latest version of PouchContainer.

``` bash
# update the apt package index
sudo apt-get update
sudo apt-get install pouch
```

After installing PouchContainer, the `pouch` group is created, but no users are added to the group.

**4. Start PouchContainer**

``` bash
sudo service pouch start
```

**5. Verify the installation**

``` bash
sudo pouch pull hello-world
sudo pouch run hello-world
```

If you cann't connect to registry.hub.docker.com, you can use the mirrors.

``` bash
sudo pouch pull registry.docker-cn.com/library/hello-world
sudo pouch run hello-world
```

### 3. Run Ubuntu with PouchContainer

**1. Pull the image of Ubuntu**

``` bash
sudo pouch pull ubuntu
```

**2. Run the image of Ubuntu**
Run ubuntu in interactive mode, and allocate a pseudo-TTY:

``` bash
sudo pouch run -it ubuntu
```