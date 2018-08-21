# PouchContainer 与 Dragonfly


容器技术有助于IT操作和维护，但是同时，镜像分发带来了巨大挑战。镜像可能有几个 Gibs的大小，拉镜像会非常慢，更不用说同时发送多个拉取请求或网络带宽低的情形。 Dragonfly此时可以发挥重要作用，作为一种P2P技术，它提供了非常高效的分发，避免了镜像分发成为容器技术的瓶颈。

## 什么是 Dragonfly

[Dragonfly](https://github.com/alibaba/Dragonfly#installation) 是一个基于P2P的文件分发系统。它解决了大规模文件分发场景下分发耗时、成功率低、带宽浪费等难题。Dragonfly 显著提升了诸如数据预热、大规模容器镜像分发等业务能力。
### 构成

#### 服务器

服务器由集群管理器组成，它从源代码下载文件并构建P2P网络。

#### 客户端

客户端分为两部分: dfget 和 proxy。dfget是一个用于下载文件的主机端工具，proxy用于截获 http 请求并将它们路由到 dfget。

## 安装Dragonfly


1.安装服务器

服务器可以分别以在物理机上或在容器上这两种方式部署， [安装Dragonfly服务器](https://github.com/alibaba/Dragonfly/blob/master/docs/install_server.md) 的步骤很简单。

2.安装客户端
- 下载[客户端软件包](https://github.com/alibaba/Dragonfly/blob/master/package/df-client.linux-amd64.tar.gz)。
- 解压缩包`tar xzvf df-client.linux-amd64.tar.gz -C /usr/local/bin`，或者你可以解压缩到你喜欢的任何目录，但记得将客户端路径添加到PATH环境中，`export PATH=$PATH:/usr/local/bin/df-client`。
- 将服务器 ip 添加到客户端配置文件中，`/etc/dragonfly.conf`，nodeIp 是你部署服务器的主机 ip。


```
[node]
address=nodeIp1,nodeIp2,...
```

关于详细安装信息，您可以在 [安装Dragonfly客户端](https://github.com/alibaba/Dragonfly/blob/master/docs/install_client.md)中找到。
## 运行 PouchContainer with dragonfly

1.启动Dragonfly代理

启动Dragonfly代理， `/usr/local/bin/df-client/df-daemon  --registry https://reg.docker.alibaba-inc.com`, 你可以添加 `--verbose` 标志来获取调试日志。Dragonfly日志可以在目录 `~/.small-dragonfly/logs`中找到。

您可以在[Dragonfly使用](https://github.com/alibaba/Dragonfly/blob/master/docs/usage.md)中找到更多的Dragonfly使用信息。

2.在PouchContainer配置文件中添加以下配置`/etc/pouch/config.json`

```
{
    "image-proxy": "http://127.0.0.1:65001"
}
```

3.拉镜像 `reg.docker.alibaba-inc.com/base/busybox:latest`, 您可以在 `~/.small-dragonfly/logs/dfdaemon.log`中找到以下输出，这意味着Dragonfly安装成功。

```
time="2018-03-06 20:08:00" level=debug msg="pre access:http://storage.docker.aliyun-inc.com/docker/registry/v2/blobs/sha256/1b/1b5110ff48b0aa7112544e1666cc7199f812243ded4128f0a1b2be027c7    38bec/data?Expires=1520339335&OSSAccessKeyId=LTAIfYaNrksx0ktL&Signature=fVEYIQzIaXyqIcAhypbmzaUx5x8%3D"
time="2018-03-06 20:08:00" level=debug msg="post access:http://storage.docker.aliyun-inc.com"
time="2018-03-06 20:08:00" level=debug msg="pre access:http://storage.docker.aliyun-inc.com/docker/registry/v2/blobs/sha256/98/9869810a78644038c8d37a5a3344de0217cb37bcc2caa2313036c6948b0    ee8da/data?Expires=1520339335&OSSAccessKeyId=LTAIfYaNrksx0ktL&Signature=Tx7JYU07Gap8RfasvCe0JGAUCo4%3D"
```