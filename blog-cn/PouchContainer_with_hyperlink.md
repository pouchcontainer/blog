# PouchContainer 文档



PouchContainer文档用来记录有关PouchContainer的一切。如果你只是对这个项目比较感兴趣，阅读这个README.md文档足以了解你期待的一切。而如果你希望更深入利用PouchContainer并发掘它更多的价值，我们的文档也将会为你提供强大的支持。



这篇文档的结构如下：

[TOC]

## 特性

[特性](https://github.com/alibaba/pouch/tree/master/docs/features)包括PouchContainer的所有特性。PouchContainer旨在为用户应用提供满意的环境，因此推出了大量特性来满足不同用户的需求，如强隔离机制（包括基于Hyper技术的容器和基于Linux容器文件系统的资源视图隔离），富容器和P2P映像分发等。PouchContainer还具有许多其他特性，如保障运输层安全性，支持Prometheus，这些特性有利于其适应不同企业的各种工作需求。对于每一个特性，我们都给出了其所适用的最佳场景。



## 开始

如果你希望尝试PouchContainer，那么[开始文档](https://github.com/alibaba/pouch/blob/master/INSTALLATION.md)正是你所需要的。这个文档介绍了预备要求、安装指南以及开始体验PouchContainer的说明。



## 命令行

对于大多数用户而言，[命令行文档](https://github.com/alibaba/pouch/tree/master/docs/commandline)是一份重要的参考资料。此处对于PouchContainer CLI中的命令给出了很详尽的解释，包括命令的说明，大意，用例和参数选项。PouchContainer使用自动化文档生成技术，因此可以保证这些命令行文档随时和源代码的保持高度一致。



## API

命令行是延展PouchContainer功能最简单的方式，但延展能力可能仍受限制。因此，对于更高需求的用户，我们在此给出API以提供更强大的延展能力。



### HTTP API

[HTTP-API.md](https://github.com/alibaba/pouch/blob/master/docs/api/HTTP_API.md)文档包括所有的HTTP API原始接口的细节。对于PouchContainer的HTTP API，我们将其全部记录在[swagger.yml](https://github.com/alibaba/pouch/blob/master/apis/swagger.yml)。由于HTTP_API.md也是由swagger.yml自动生成的，因此可以保证代码实现和HTTP API文档的高度一致。



### HTTP API 更新日志

[HTTP-API-CHANGELOG.md](https://github.com/alibaba/pouch/blob/master/docs/api/HTTP_API_CHANGELOG.md)文档包括了所有HTTP API的更新日志。原则上来说，PouchContainer的HTTP API应该始终保持不变，不论经历多少代更新。然而，随着PouchContainer的迭代进化，一些关于API的功能添加，方向调整和漏洞修复可能会导致HTTP API的一些变化。因此，我们使用更新日志来记录追踪这一切的变化。



### gRPC API

[GRPC-API.md](https://github.com/alibaba/pouch/blob/master/docs/api/GRPC_API.md)文档包括所有原始的gRPC API的细节。PouchContainer可以被当做原生的容器引擎来运行，以支持CRI。此处，我们使用gRPC来暴露CRI能力。



### gRPC API 更新日志

[GRPC-API-CHANGELOG.md](https://github.com/alibaba/pouch/blob/master/docs/api/GRPC_API_CHANGELOG.md)文档包括了所有gRPC API的更新日志。原则上来说，PouchContainer的gRPC API应该始终保持不变，不论经历多少代更新。然而，随着PouchContainer的迭代进化，一些关于API的功能添加，方向调整和漏洞修复可能会导致gRPC API的一些变化。因此，我们使用更新日志来记录追踪这一切的变化。



## Kubernetes

[Kubernetes](https://github.com/alibaba/pouch/tree/master/docs/kubernetes)记录了PouchContainer中所有和Kubernetes相关的内容。在PouchContainer设计伊始，整个团队决定将其定位为对于Kubernetes的可靠运行选项（runtime option）。为了支持Kubernetes，PouchContainer直接实现了容器运行接口（CRI）。该文件夹包括了对于Kubernetes和PouchContainer的介绍，如何开始结合二者进行使用，如何测试以及有关这二者的架构。



## 测试指南

[测试指南](https://github.com/alibaba/pouch/tree/master/docs/test)可以很好地帮助贡献开发者们了解并搭建测试环境。目前，我们可以将PouchContainer的测试分为四个维度：单元测试，API集成测试，CLI集成测试和CRI有效性测试。更多的细节请参看[测试指南](https://github.com/alibaba/pouch/tree/master/docs/test)。



## 贡献

[贡献文档](https://github.com/alibaba/pouch/tree/master/docs/contributions)将告知社区参与者如何对PouchContainer项目做出贡献以及如何在这个项目中进行协作互助。对于所有的参与者，我们给出了一些指南和友情提醒。



## 底层技术

[底层技术](https://github.com/alibaba/pouch/tree/master/docs/underlying_tech)帮助用户们了解更多PouchContainer中所采用的底层技术，如Linux内核中的Cgroup与命名空间技术，虚拟管理（hypervisor）容器技术以及Linux容器文件系统等。如果用户希望更好掌握PouchContainer的相关技术，请务必阅读这些文档。



## 静态文件

[静态文件](https://github.com/alibaba/pouch/tree/master/docs/static_files)包含了PouchContainer中使用的所有静态文件。如果你正在寻找一些特别的东西，比如PouchContainer的日志，或者其架构，再或者一些相关的图片，你可能能在这里找到答案。