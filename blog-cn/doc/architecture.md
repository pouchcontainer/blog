##  结构

&ensp; &ensp; &ensp; 为了阐明PouchContainer在容器系统里的定位，我们使用了一个清晰的结构来打造它。为了确保功能上的强隔离性，用几个部分来组成PouchContainer。说到结构，主要包括以下两部分：

* 生态系统结构

* 组成结构

### 生态系统结构

​    在PouchContainer's的技术路线中，我们把拥抱生态系统设置为一个主要目标。对于上部的编配层来说，PouchContainer 支持Kubernetes和Swarm，对于下部的运行时层来说PouchContainer支持使用例如runC、runV、runlxc等来运行容器。CNI和CSI使得存储和网络有能力提供大容量的供给。

  ![avatar](https://raw.githubusercontent.com/alibaba/pouch/master/docs/static_files/pouch_ecosystem_architecture_no_logo.png)

  初见时，可能会觉得生态系统结构有些复杂，不过别担心，以下三个维度的介绍将使你对它有一个全面的理解。

  ### 容器运行时层

  容器运行时层位于结构图的右上方，这方面主要关注PouchContainer支持的兼容OCI的运行，这些运行时为操作系统进程和应用容器统一了规范。目前PouchContainer支持以下四种运行时：

  * runC

  * runlxc

  * runV

  * clear containers

  使用runC可以使PouchContainer像以docker为例的其他容器引擎一样生成普通容器。使用runlxc可以使PouchContainer生成基于LXC的容器。当用户需要在能够兼容2.6.32以上的内核运行容器时，runlxc就能大展身手了。基于系统管理程序的容器也有许多应用场景，使用Pouchcontainer，runV和clear container可以实现。
  本文提到的四种运行时都支持containerd，containerd接管了所有的包括创建、开始、停止、删除等等的容器管理。
#### 编配层
从Pouchcontainer诞生之日起，它总是积极地支持Kubernetes。我们在结构图的上半部分说明了这一点。首先，Pouchcontainer将在内部集成cri-containerd，所以Kubernetes可以简单地控制Pouchcontainer来管理Pod。工作流将流经cri-containerd，containerd客户端，containerd，runC/runV和pod。cri-containerd将利用实现CNI接口的网络插件优势，设定Pod网络配置。(注：此处的implement应为implements)。
#### 容器层
我们不仅支持Kubernetes里的Pod，也为用户提供简单的容器管理。这对开发者来说十分实用。换句话说，Pouchcontainer支持单一容器API。在这种情况下，工作流流过pouchd，containerd客户端，containerd，runC/runV和容器。从网络层面看，Pouchcontainer使用libnetwork构造容器的网络。不仅如此，lxcfs也用来保证容器之间的隔离以及容器和主机之间的隔离。
### 组成结构 
***
Pouchcontainer的生态系统结构向我们展示了它在容器生态系统中的定位。下面的图片表明了Pouchcontainer的组成结构。在组成结构中，我们将Pouchcontainer分为两个主要部分：Pouchcontainer CLI和Pouchd。
![avatar](https://raw.githubusercontent.com/alibaba/pouch/master/docs/static_files/pouch_component_architecture.png)
​    

#### Pouchcontainer CLI

以下是囊括在Pouchcontainer CLI里的不同命令，例如产生、开始、执行等等。用户可以使用Pouchcontainer CLI和Pouchd。当执行命令时，PouchContainer CLI将把它转化到Pouchd API的回调来满足用户需求。PouchContainer客户端API在PouchContainer CLI里已经被包装好。其他人可以很轻松地把PouchContainer客户端包集成到第三方软件里，这个包只支持Go语言。当通过PouchContainer客户端包调用Pouchd时，整个流程是基于HTTP的。

#### Pouchd

Pouchd从一开始就设计为解耦式，使得Pouchd易于理解。并且有助于破解PouchContainer。通常来说，我们认为Pouchd可以分为以下几部分： 

* HTTP服务器 

* 桥接层

* 管理器（系统/网络/容量/集装箱/镜像）

* ctrd
HTTP 服务器直接接收API调用并回复客户端。它的工作是解析请求，构造可传递给桥接层的正确结构，并且无论服务器否成功处理请求，都会构建响应。 
桥接层是一个转换层，它处理来自客户端的对象以满足管理器或容器的需求，并处理来自管理器和containerd的对象，以使响应与Moby的API兼容。
管理器 是Pouchd的主要处理者。它能从请求中处理合适的object，并完成对应的工作。 Pouchd目前有五个管理器：容器管理器，镜像管理器，网络管理器，容量管理器和系统管理器。
ctrd是Pouchd的containerd客户端。使用Ctrd可以正确地使manager与containerd建立通信。管理器在ctrd中调用函数并向containerd发送请求。此外，当容器状态发生变化时，containerd能够最先探测到，用容器监视goroutines来检测，并更新存储在缓存中的内部数据。









