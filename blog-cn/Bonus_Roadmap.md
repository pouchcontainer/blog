# 路线图

路线图对于PouchContainer项目中高优先级的项目提供了详尽的描述。这有利于PouchContainer的贡献者们更好了解项目的进化的方向，以及确定潜在的贡献是否会偏离主方向。



如果一个特性没有被列举在下面，这并不说明我们不考虑这个特性。我们始终热烈欢迎所有的贡献。但是请理解这种特性可能会需要开发者更多的时间进行审核。



我们在路线图中设计了三个部分：

- 容器常规管理
- 强隔离
- 对于生态系统的开放



## 容器常规管理

我们希望在容器管理方面给用户焕然一新的体验，因此我们将这部分作为第一步。[Moby](https://github.com/moby/moby)使得工业容器API标准流行起来。我们的PouchContainer会遵循这个API标准来提供容器服务。PouchContainer会更加注意多方面的情况，以使得容器可以在多种隔离单元之上运行。同时，我们也希望为用户提供更好的应用维护体验。



## 强隔离

为了提高容器的工业安全性，我们做了很多工作，但是容器技术还没有达到预期。PouchContainer将会在强隔离领域，无论在软件端还是硬件端，发挥更多的作用。鉴于安全是限制这项技术应用于生产环境的最大阻碍，PouchContainer会在以下领域提升隔离性能：以用户空间的Linux容器文件系统来隔离资源视图，基于虚拟管理的容器和基于kvm的容器等。



## 强化生态系统

为了面向容器生态系统的开放，PouchContainer的扩展性将会被纳入设计。作为一个容器引擎，PouchContainer将会支持pod，并且可以对于上层编配层与[Kubernetes](https://github.com/kubernetes/kubernetes)进行集成。在基础架构管理方面，PouchContainer将支持[CNI](https://github.com/containernetworking/cni)和[CSI](https://github.com/container-storage-interface)。在监控，日志和许多其他领域，PouchContainer都在努力面向原生云进行开放。