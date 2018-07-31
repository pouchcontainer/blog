# 具有Dragonfly的PouchContainer

容器技术有利于促进IT操作和维护，但同时为图像的分布带来了巨大的挑战。图像可以像几个Gibs一样大，拉动图像会非常慢，更不用说多个拉动请求或低网络带宽的时候。Dragonfly在这里可以发挥重要作用，作为一种P2P技术，它提供了非常高效的分布，避免了图像分发成为容器技术的瓶颈。

## 什么是Dragonfly

[Dragonfly](https://github.com/alibaba/Dragonfly#installation) 是一个P2P文件分布系统。它解决了大规模文件分发场景中耗时分布，低成功率和浪费带宽的问题。 Dragonfly显着提高了数据预热和大规模容器图像分发等服务功能。

### 组成

#### 服务器

服务器由集群管理器组成，它从源代码下载文件并构建P2P网络。

#### 客户端

客户端包含了两部分：dfget和proxy。Dfget是一种用于下载文件的主机端工具，proxy用于拦截http请求并将它们发送到dfget。

## 安装dragonfly

1.安装服务器

服务器可以两种方式部署，在物理机器上或在容器上，[install dragonfly server](https://github.com/alibaba/Dragonfly/blob/master/docs/install_server.md)的步骤很简单。

2.安装客户端

- 下载[client package](https://github.com/alibaba/Dragonfly/blob/master/package/df-client.linux-amd64.tar.gz).
- 解压缩包`tar xzvf df-client.linux-amd64.tar.gz -C /usr/local/bin`，或者你可以解压到任何你喜欢的目录，但是记得将客户端路径添加到PATH环境中， `export PATH=$PATH:/usr/local/bin/df-client`。
- 添加服务端ip到客户端配置文件中， `/etc/dragonfly.conf`，nodeIp是部署服务器主机的IP。

```
[node]
address=nodeIp1,nodeIp2,...
```

关于详细的安装信息，可以从 [install dragonfly client](https://github.com/alibaba/Dragonfly/blob/master/docs/install_client.md)中获得。

## 运行具有dragonfly的PouchContainer

1.开启dragonfly代理

开启dragonfly代理，`/usr/local/bin/df-client/df-daemon  --registry https://reg.docker.alibaba-inc.com`，你可以增加标志 `--verbose` 来获取debug日志。 Dragonfly日志可在文件`~/.small-dragonfly/logs`中找到。

更多的dragonfly用法信息可在[dragonfly usage](https://github.com/alibaba/Dragonfly/blob/master/docs/usage.md)中找到。

2.添加下面的配置到PouchContain配置文件`/etc/pouch/config.json`中

```
{
    "image-proxy": "http://127.0.0.1:65001"
}
```

3.拉取一个图像`reg.docker.alibaba-inc.com/base/busybox:latest`，你可在`~/.small-dragonfly/logs/dfdaemon.log`中找到下列输出，这意味着dragonfly正在工作。

```
time="2018-03-06 20:08:00" level=debug msg="pre access:http://storage.docker.aliyun-inc.com/docker/registry/v2/blobs/sha256/1b/1b5110ff48b0aa7112544e1666cc7199f812243ded4128f0a1b2be027c7    38bec/data?Expires=1520339335&OSSAccessKeyId=LTAIfYaNrksx0ktL&Signature=fVEYIQzIaXyqIcAhypbmzaUx5x8%3D"
time="2018-03-06 20:08:00" level=debug msg="post access:http://storage.docker.aliyun-inc.com"
time="2018-03-06 20:08:00" level=debug msg="pre access:http://storage.docker.aliyun-inc.com/docker/registry/v2/blobs/sha256/98/9869810a78644038c8d37a5a3344de0217cb37bcc2caa2313036c6948b0    ee8da/data?Expires=1520339335&OSSAccessKeyId=LTAIfYaNrksx0ktL&Signature=Tx7JYU07Gap8RfasvCe0JGAUCo4%3D"
```
