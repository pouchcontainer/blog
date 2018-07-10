### *original link: [ROADMAP.md](https://github.com/alibaba/pouch/blob/master/ROADMAP.md)*

# 路线图
路线图提供了PouchContainer决定的优先考虑的事项的细节描述。它帮助PouchContainer项目参与者更好的理解项目的发展方向并帮助判断对项目的贡献是否符合该方向。

如果某一条特性没有被包括在下表中，这并不意味着我们不会对这条特性给予考虑。我们一直欢迎任何的贡献。但请理解这样的贡献可能需要审核人员更多的时间去审查。

> committer处是否应该是审查人员（reviewer或者maintainer）而非提交人员？

我们在路线图中设计了三个部分：
* 容器一般管理
* 强隔离
* 向生态开放

## 容器一般管理
我们将优化用户在管理容器的过程体验作为第一步，也是非常重要的一步。[Moby](https://github.com/moby/moby)在业界有非常流行的容器API标准。PouchContainer会遵守这个API标准来提供容器服务。另外，在如何在不同隔离单元之上运行容器这方面，PouchContainer会投入更大的精力。同时，我们也会致力于在应用管理上如何使用户获得更好的体验。

## 强隔离
在业界，在提升容器安全性这方面已经有很多人做了很多工作，但是容器科技还没有达到预期的安全标准。在软件和硬件两方面，PouchContainer都会更多地使用强隔离来提升安全性。因为安全性问题目前是技术产业化道路上最大的障碍，PouchContainer会在如下几个方面提高隔离性：使用userspace lxcfs来隔离资源视图，基于虚拟机的容器以及基于KVM的容器等等。

## 向生态开放
为了向容器生态开放，PouchContainer会被设计成可扩展的。作为容器引擎，PouchContainer将支持pod并且将能够和[kubernetes](https://github.com/kubernetes/kubernetes)的编排层上层进行整合。对于基础设施管理来说，PouchContainer会实现[CNI](https://github.com/containernetworking/cni)和[CSI](https://github.com/container-storage-interface)接口。在监控，日志等方面，PouchContainer会在实现云原生道路上扮演一个开放的角色。