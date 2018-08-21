[origin doc link](https://github.com/alibaba/pouch/blob/master/ROADMAP.md)
# 项目路线图

该路线图详细描述了PouchContainer项目将优先处理的事务。这有助于PouchContainer的贡献者更多地了解PouchContainer的发展方向，以及判断自己将要提的贡献是否偏离了方向。

可能有些功能并没有在下面被列出，但这并不意味着他们永远都不会被考虑在内。我们说过我们热烈欢迎大家提出的所有贡献。但请理解，有些贡献可能需要更多时间以便提交者进行审核。

我们在路线图中设计了三个部分：

* 容器定期管理
* 强隔离
* 对生态系统的开放

## 容器定期管理

我们将提升容器管理的用户体验作为最重要的一步。[Moby](https://github.com/moby/moby)已经在行业中推广了容器的API标准，并且PouchContainer也将遵循此API标准来提供容器服务。此外，PouchContainer将更多关注如何在各种隔离单元上运行容器方面的内容。同时，如果在看管应用程序方面有更好的经验，我们也将欣然接受。

## 强隔离

虽然业界已经在提高容器安全性方面做了很多工作，但容器技术尚未达到我们想要的目标。因此无论是在软件还是硬件方面，PouchContainer都将对强隔离产生更大的影响。由于安全性是将技术应用于生产环境的最大壁垒，PouchContainer将提高以下方面的隔离能力：以用户空间lxcfs来隔离资源视图，基于hypervisor的容器，基于kvm的容器等。

## 增强生态系统建设

为了对容器生态系统开放PouchContainer，PouchContainer将被设计为具有可扩展性。作为容器引擎，PouchContainer将支持pod并能够将上层编排层的[Kubernetes](https://github.com/kubernetes/kubernetes)相集成。对于基础设施，PouchContainer将采用[CNI](https://github.com/containernetworking/cni)和[CSI](https://github.com/container-storage-interface)对其进行管理。在监控和日志记录等方面，PouchContainer作为一个开放性的角色来解决Cloud Native。
