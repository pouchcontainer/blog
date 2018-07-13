# Roadmap

Roadmap详细说明了PouchContainer项目决定有限考虑的条目。它有助于PouchContainer的贡献者更好的理解项目的发展方向以及接下来可能要做的贡献是否偏离了这个方向。

如果某些特性没有被列在下面，这并不意味着我们不会考虑它们。我们总是再说，任何形式的贡献我们都十分欢迎。但是请理解那些没有被列出的贡献可能会花费提交者更多的时间去评审。

我们在设计Roadmap时涉及了三个方面：

* 容器定期管理
* 增强隔离
* 开放生态系统

## 容器定期管理

我们会将提升用户对于容器管理方面的体验作为最重要的一步。[Moby](https://github.com/moby/moby)已经将容器API标准在业内进行普及。PouchContainer会遵循这些API标准来提供容器服务。此外，PouchContainer会更加多方面的思考如何在多种隔离单元上运行容器。提升在处理应用程序上的体验也是我们需要思考的一部分。

## 增强隔离

业内已经做了很多工作去提升容器的安全性。但是容器技术还没达到所期待的目标。PouchContainer会在强隔离上采取更多的行动，不论是在软件层面还是硬件层面。在生产环境中应用技术的最大障碍就是安全，因此PouchContainer会在以下几个领取提升其隔离能力：隔离资源视图的userspace lxcfs，基于容器的管理程序，基于kvm的容器等等。

## 开放生态系统

我们想要让PouchContainer可以开放给容器生态系统，于是就把它设计为可扩展的。作为一个容器引擎，PouchContainer可以支持pod，并且可以通过 [kubernetes](https://github.com/kubernetes/kubernetes) 集成上层编排层（upper orchestration layer）。对于基本的基础设施管理，PoucnContainer可以接受 [CNI](https://github.com/containernetworking/cni) 和 [CSI](https://github.com/container-storage-interface)。而在监控、日志记录等方面，PouchContainer则对于原生云的探讨发挥了开放的作用




