### 1. Brief introduction to CRI

At the bottom of each Kubernetes node, a program is responsible for the creation and deletion of specific container, and Kubernetes calls its interface to complete the container scheduling. We call this layer of software the Container Runtime, which is represented by the famous Docker.

Of course, Docker is not the only container runtime, including the RKT of CoreOS, the runV of hyper.sh, the gvisor of Google, and the PouchContainer of this article. All of them contain complete container operations that can be used to create containers with different characteristics. Different kinds of container runtime have their own unique advantages and can meet the needs of different users. Therefore, it is imperative for Kubernetes to support multiple container runtimes.

Initially, Kubernetes had a built-in call interface to Docker, and then the community integrated the RKT interface in Kubernetes 1.3, making it an optional container runtime in addition to Docker. However, at this time, both calls to Docker and to RKT are strongly coupled to Kubernetes' core code, which undoubtedly brings the following two problems:

1. Emerging container operators, such as PouchContainer, are struggling to add Kubernetes to their ecosystem. Developers of the container runtime must have a very deep understanding of Kubernetes's code (at least Kubelet) in order to successfully connect the two.
2. Kubernetes' code will be more difficult to maintain, which is reflected in two aspects:（1）If hard-coding all the call interfaces of the various container runtime into Kubernetes, the core code of Kubernetes will be bloated,（2）Minor changes to the container runtime interface will trigger changes to the core code of Kubernetes and increase its instability.

In order to solve these problems, the community introduced the Container Runtime Interface in Kubernetes 1.5. By defining a set of common interfaces of the Container Runtime, the calling Interface of Kubernetes for various container runtimes was shielded from the core code. The core code of Kubernetes only called the abstract Interface layer. However, for various containers, Kubernetes can be accessed smoothly as long as the interfaces defined in CRI are satisfied, which makes it one of the container runtime options. The solution, while simple, is a liberation for the Kubernetes community maintainers and container runtime developers.

### 2. CRI设计概述



![cri-1.png | left | 827x299](https://cdn.yuque.com/lark/0/2018/png/103564/1527478355304-a20865ae-81b8-4f13-910d-39c9db4c72e2.png "")


如上图所示，左边的Kubelet是Kubernetes集群的Node Agent，它会对本节点上容器的状态进行监控，保证它们都按照预期状态运行。为了实现这一目标，Kubelet会不断调用相关的CRI接口来对容器进行同步。

CRI shim则可以认为是一个接口转换层，它会将CRI接口，转换成对应底层容器运行时的接口，并调用执行，返回结果。对于有的容器运行时，CRI shim是作为一个独立的进程存在的，例如当选用Docker为Kubernetes的容器运行时，Kubelet初始化时，会附带启动一个Docker shim进程，它就是Docker的CRI shime。而对于PouchContainer，它的CRI shim则是内嵌在Pouchd中的，我们将其称之为CRI manager。关于这一点，我们会在下一节讨论PouchContainer相关架构时再详细叙述。

CRI本质上是一套gRPC接口，Kubelet内置了一个gRPC Client，CRI shim中则内置了一个gRPC Server。Kubelet每一次对CRI接口的调用，都将转换为gRPC请求由gRPC Client发送给CRI shim中的gRPC Server。Server调用底层的容器运行时对请求进行处理并返回结果，由此完成一次CRI接口调用。

CRI定义的gRPC接口可划分两类，ImageService和RuntimeService：其中ImageService负责管理容器的镜像，而RuntimeService则负责对容器生命周期进行管理以及与容器进行交互（exec/attach/port-forward）。

### 3. CRI Manager架构设计



![yzz's pic.jpg | left | 827x512](https://cdn.yuque.com/lark/0/2018/jpeg/95844/1527582870490-a9b9591d-d529-4b7d-bc5f-69514ef115e7.jpeg "")


在PouchContainer的整个架构体系中，CRI Manager实现了CRI定义的全部接口，担任了PouchContainer中CRI shim的角色。当Kubelet调用一个CRI接口时，请求就会通过Kubelet的gRPC Client发送到上图的gRPC Server中。Server会对请求进行解析，并调用CRI Manager相应的方法进行处理。

我们先通过一个例子来简单了解一下各个模块的功能。例如，当到达的请求为创建一个Pod，那么CRI Manager会先将获取到的CRI格式的配置转换成符合PouchContainer接口要求的格式，调用Image Manager拉取所需的镜像，再调用Container Manager创建所需的容器，并调用CNI Manager，利用CNI插件对Pod的网络进行配置。最后，Stream Server会对交互类型的CRI请求，例如exec/attach/portforward进行处理。

值得注意的是，CNI Manager和Stream Server是CRI Manager的子模块，而CRI Manager，Container Manager以及Image Manager是三个平等的模块，它们都位于同一个二进制文件Pouchd中，因此它们之间的调用都是最为直接的函数调用，并不存在例如Docker shim与Docker交互时，所需要的远程调用开销。下面，我们将进入CRI Manager内部，对其中重要功能的实现做更为深入的理解。

### 4. Pod模型的实现

在Kubernetes的世界里，Pod是最小的调度部署单元。简单地说，一个Pod就是由一些关联较为紧密的容器构成的容器组。作为一个整体，这些“亲密”的容器之间会共享一些东西，从而让它们之间的交互更为高效。例如，对于网络，同一个Pod中的容器会共享同一个IP地址和端口空间，从而使它们能直接通过localhost互相访问。对于存储，Pod中定义的volume会挂载到其中的每个容器中，从而让每个容器都能对其进行访问。

事实上，只要一组容器之间共享某些Linux Namespace以及挂载相同的volume就能实现上述的所有特性。下面，我们就通过创建一个具体的Pod来分析PouchContainer中的CRI Manager是如何实现Pod模型的：

1. 当Kubelet需要新建一个Pod时，首先会对`RunPodSandbox`这一CRI接口进行调用，而CRI Manager对该接口的实现是创建一个我们称之为"infra container"的特殊容器。从容器实现的角度来看，它并不特殊，无非是调用Container Manager，创建一个镜像为`pause-amd64:3.0`的普通容器。但是从整个Pod容器组的角度来看，它是有着特殊作用的，正是它将自己的Linux Namespace贡献出来，作为上文所说的各容器共享的Linux Namespace，将容器组中的所有容器联结到一起。它更像是一个载体，承载了Pod中所有其他的容器，为它们的运行提供基础设施。而一般我们也用infra container代表一个Pod。
2. 在infra container创建完成之后，Kubelet会对Pod容器组中的其他容器进行创建。每创建一个容器就是连续调用`CreateContainer`和`StartContainer`这两个CRI接口。对于`CreateContainer`，CRI Manager仅仅只是将CRI格式的容器配置转换为PouchContainer格式的容器配置，再将其传递给Container Manager，由其完成具体的容器创建工作。这里我们唯一需要关心的问题是，该容器如何加入上文中提到的infra container的Linux Namespace。其实真正的实现非常简单，在Container Manager的容器配置参数中有`PidMode`, `IpcMode`以及`NetworkMode`三个参数，分别用于配置容器的Pid Namespace，Ipc Namespace和Network Namespace。笼统地说，对于容器的Namespace的配置一般都有两种模式："None"模式，即创建该容器自己独有的Namespace，另一种即为"Container"模式，即加入另一个容器的Namespace。显然，我们只需要将上述三个参数配置为"Container"模式，加入infra container的Namespace即可。具体是如何加入的，CRI Manager并不需要关心。对于`StartContainer`，CRI Manager仅仅只是做了一层转发，从请求中获取容器ID并调用Container Manager的`Start`接口启动容器。
3. 最后，Kubelet会不断调用`ListPodSandbox`和`ListContainers`这两个CRI接口来获取本节点上容器的运行状态。其中`ListPodSandbox`罗列的其实就是各个infra container的状态，而`ListContainer`罗列的是除了infra container以外其他容器的状态。现在问题是，对于Container Manager来说，infra container和其他container并不存在任何区别。那么CRI Manager是如何对这些容器进行区分的呢？事实上，CRI Manager在创建容器时，会在已有容器配置的基础之上，额外增加一个label，标志该容器的类型。从而在实现`ListPodSandbox`和`ListContainers`接口的时候，以该label的值作为条件，就能对不同类型的容器进行过滤。

综上，对于Pod的创建，我们可以概述为先创建infra container，再创建pod中的其他容器，并让它们加入infra container的Linux Namespace。

### 5. Pod Network Configuration

Since all containers inside Pod share a Network Namespace, we only need to configure Network Namespace when creating the infra container.

All network functions of containers in Kubernetes ecosystem are implemented via CNI. Similar to CRI, CNI is a set of standard interfaces and any network schema implementing these interfaces can be integrated into Kubernetes seamlessly. CNI Manager in CRI Manager is a simple encapsulation of CNI. It will load configuration files under `/etc/cni/net.d` during initialization, as follows:

```sh
$ cat >/etc/cni/net.d/10-mynet.conflist <<EOF
{
        "cniVersion": "0.3.0",
        "name": "mynet",
        "plugins": [
          {
                "type": "bridge",
                "bridge": "cni0",
                "isGateway": true,
                "ipMasq": true,
                "ipam": {
                        "type": "host-local",
                        "subnet": "10.22.0.0/16",
                        "routes": [
                                { "dst": "0.0.0.0/0" }
                        ]
                }
          }
        ]
}
EOF
```

In which the CNI plugins used to configure Pod network are specified, such as the `bridge` above, as well as some network configurations, like the subnet range and routing configuration of Pod on current node.

Now let's show how to register a Pod into CNI step by step:

1. When creating a infra container via container manager, set `NetworkMode` as "None", indicating creating a unique Network Namespace owned by this infra container without any configuration.
2. Based on the corresponding PID of infra container, the directory of corresponding Network Namespace (`/proc/{pid}/ns/net`) can be obtained.
3. Invoke `SetUpPodNetwork` method of CNI Manager, the core argument is the Network Namespace directory obtained in step 2. This method in turn invokes CNI plugins specified during the initialization of CNI Manager, such as 'bridge' in above snippet, configuring Network Namespace specified in argumants, including creating various network devices, configuring network, registering this Network Namespace into corresponding CNI network.

For most Pods, network configuration can be done through the above steps and most of the work are done by CNI and corresponding plugins. But for some special Pods, whose network mode are set to "Host", that is, sharing Network Namespace with host machine. In this case, we only need to set `NetworkMode` to "Host" when creating infra container via Container Manager, and skip the configuration of CNI Manager.

For other containers in a Pod, either the Pod is in "Host" mode, or it has independent Network Namespace, we only need to set `NetworkMode` to "Container" when creating containers via Container Manager, and join the Network Namespace that infra container belongs to.

### 6. IO stream processing

Kubernetes provides features such as `kubectl exec/attach/port-forward` to enable direct interaction between the user and a specific Pod or container. The command are as follows:

```sh
aster $ kubectl exec -it shell-demo -- /bin/bash
root@shell-demo:/# ls
bin   dev  home  lib64  mnt  proc  run   srv  tmp  var
boot  etc  lib   media  opt  root  sbin  sys  usr
root@shell-demo:/#
```
As we can see, `exec` a Pod is equivalent to `ssh` logging into the container. Below, we analyze the processing of IO requests in Kubernetes in the execution flow of `kubectl exec` and the role that CRI Manager plays.



![stream-3.png | left | 827x296](https://cdn.yuque.com/lark/0/2018/png/103564/1527478375654-1c891ac5-7dd0-4432-9f72-56c4feb35ac6.png "")


As shown above, the steps to execute a kubectl exec command are as follows:

1.The essence of the `kubectl exec` command is to execute an exec command on a container in the Kubernetes cluster and forward the resulting IO stream to the users. So the request will first be forwarded to the Kubelet of the node where the container is located, and the Kubelet will then call the `Exec` interface in the CRI according to the configuration. The requested configuration parameters are as follows:

```go
type ExecRequest struct {
    ContainerId string  // execute the aimed container of exec
    Cmd []string    // specific execution of the exec command
    Tty bool    // whether to execute the exec command in a TTY
    Stdin bool  // whether to include Stdin flow
    Stdout bool // whether to include Stdout flow
    Stderr bool // whether to include Stderr flow
}
```

2.Surprisingly, CRI Manager's `Exec` method does not directly call Container Manager, and executes the exec command on the target container, but instead calls the built-in Stream Server `GetExec` method.

3.The work done by the Stream Server `GetExec` method is to save the contents of the exec request to the Request Cache shown in the figure above, and return a token. With this token, we can retrieve the corresponding exec request from the Request Cache. Finally, the token is written to a URL and returned to the ApiServer as a result layer.

4.ApiServer uses the returned URL to directly initiate a http request to the Stream Server of the node where the target container is located. The request header contains the "Upgrade" field, which is required to upgrade the http protocol to a streaming protocol such as websocket or SPDY to support multiple IOs. Stream processing, this article we take SPDY as an example.

5.The Stream Server processes the request sent by the ApiServer. First, the previously saved exec request configuration is obtained from the Request Cache according to the token in the URL. After that, it replies to the http request, agree to upgrade the protocol to SPDY, and waits for the ApiServer to create specified numbers of streams according to the configuration of the exec request, corresponding to the standard input Stdin, the standard output Stdout, and the standard error output Stderr.

6.After the Stream Server obtains the specified number of Streams, the Container Manager's `CreateExec` and `startExec` methods are invoked in turn to perform an exec operation on the target container and forward the IO stream to the corresponding stream.

7.Finally, the ApiServer forwards the data of each stream to the user, and starts the IO interaction between the user and the target container.

In fact, before the introduction of CRI, Kubernetes handled the IO in the same way as we expected. Kubelet will execute the exec command directly on the target container and forward the IO stream back to the ApiServer. But this will put Kubelet under too much pressure, and all IO streams need to be forwarded through it, which is obviously unnecessary. Therefore, although the above processing is complicated at first glance, it effectively alleviates the pressure of Kubelet and makes the processing of IO more efficient.

### 7. 总结

本文从引入CRI的缘由而起，简要描述了CRI的架构，重点叙述了PouchContainer对CRI各个核心功能模块的实现。CRI的存在让PouchContainer容器加入Kubernetes生态变得更为简单快捷。而我们也相信，PouchContainer独有的特性必定会让Kubernetes生态变得更加丰富多彩。

<span data-type="color" style="color:rgb(38, 38, 38)"><span data-type="background" style="background-color:rgb(255, 255, 255)">PouchContainer CRI的设计与实现，是阿里巴巴-浙江大学前沿技术联合研究中心的联合研究项目，旨在帮助PouchContainer 作为一种成熟的容器运行时（container runtime），积极在生态层面拥抱 CNCF。浙江大学 SEL 实验室的卓越技术力量，有效帮助 Pouch 完成 CRI 层面的空白，未来预计在阿里巴巴以及其他使用PouchContainer的数据中心中，创造不可估量的价值。</span></span>

### 参考文献

* [Introducing Container Runtime Interface (CRI) in Kubernetes](https://kubernetes.io/blog/2016/12/container-runtime-interface-cri-in-kubernetes/)
* [CRI Streaming Requests Design Doc](https://docs.google.com/document/d/1OE_QoInPlVCK9rMAx9aybRmgFiVjHpJCHI9LrfdNM_s/edit#)
