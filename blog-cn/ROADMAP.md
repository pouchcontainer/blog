# 路线图

路线图提供了PouchContainer项目决定优化部分的详细介绍。它可以帮助PouchContainer贡献者更多的理解演化方向和一个可能的贡献是否偏离了方向。

如果一个特性没有被列在下面的列表中，并不代表我们永远不会将其纳入考虑范围。我们总是说我们热情地欢迎所有的贡献者，但是那种特性可能会令贡献者们多花费一点时间进行代码审查

我们为路线图设计了三个部分

* 容器常规管理
* 强隔离性
* 开放生态系统

## 容器常规管理

我们将会把提升用户在容器管理方面的体验作为最重要的目标。[Moby](https://github.com/moby/moby)的容器API在工业界非常流行，PouchContainer将会遵从它的API标准来提供容器服务。另外，PouchContainer也会更加关心如何在多样的隔离单元上运行容器和如何在保护应用方面提供更好的体验。

## 强隔离性

在工业界已经有很多提升容器安全性的工作，但他们还没有达到目标。PouchContainer将会从软件和硬件两方面努力提升容器的安全性。因为安全性是容器技术应用到生产环境的最大障碍，PouchContainer将会从以下方面改进容器隔离性：用于隔离资源视图的用户态lxcfs，基于hypervisor的容器，基于kvm的容器等等。


## 开放生态系统

为了开放容器生态系统，PouchContainer被设计为可扩展的。作为一个容器引擎，PouchContainer将支持pod并且通过[kubernetes](https://github.com/kubernetes/kubernetes)支持上层服务编排层的接入。为了基础设施的管理，PouchContainer将会拥抱[CNI](https://github.com/containernetworking/cni)和[CSI](https://github.com/container-storage-interface)。通过在监控，日志和其它方面的努力，PouchContainer在靠近云原生中扮演着一个开放的角色。

