[origin doc link](https://github.com/alibaba/pouch/blob/master/FAQ.md)

# 常见问题

## 什么是PouchContainer项目
PouchContainer是一个为开发和运维人员提供容器服务的工具。它可以帮助IT工程组搭建一个流畅的工作流。同时PouchContainer也可以加速企业开发运维革新。

对于开发人员，PouchContainer为他们提供了一种打包应用的标准方法。使用PounchContainer，开发人员可以毫不费力地打包自己的应用，并且可以在任何地方运行运行它。PouchContainer提供的标准化环境使运行持续集成和提高持续交付的效率变得更加容易。

对于运维人员，PouchContainer实现了自动化并且极大减少了手动操作。有了PounchContainer运维人员不在需要担心异构机器架构以及环境问题。PounchContainer可以让运维人员的注意力更多地集中在应用程序的操作。

对于数据中心所有者，PouchContainer是您的最佳选择。与VM技术相比，PouchContainer在提供了相似隔离级别的同时提高了资源利用率。

## 为什么叫它PouchContainer

PouchContainer是指某些小袋子。一种是育雏袋，用于保护非常幼小的生命。这是一个比喻，Software PouchContainer有责任非常密切地照看应用程序。换句话说，应用程序是PouchContainer世界中的关键词。

## PounchContainer的历史是什么？

始于2011年，PouchContainer最初是阿里巴巴的一个单纯的容器服务，用于服务淘宝的数百万交易业务。那时，PouchContainer基于一种叫作[LXC](https://en.wikipedia.org/wiki/LXC)的技术。

随着容器技术在工业界的发展，[Docker](https://www.docker.com/)技术应运而生，并以其创新的分层镜像技术而受到欢迎。 2015年，PouchContainer将docker的镜像技术引入其自己的架构中，使得自身更加强大。

随着越来越多的场景体验，PouchContainer获得了大量的提升，并且确实使得生产准备就绪。目前它支持阿里巴巴的大部分业务的运行。

## PouchContainer在容器生态系统中的作用是什么

也许很多人会说容器生态系统已经非常成熟了。那么PouchContainer的作用是什么呢？

首先，我们承认容器生态系统中有很多软件。然而，根据阿里巴巴的容器技术经验，虽然目前的生态系统很好，但可以更好，特别是在以应用作为容器引擎的态度上。因此PouchContainer是容器生态系统中一款更轻，更实用的容器引擎。

在容器运行时的底层支持中，PouchContainer认为基于hypervisor的量级较轻的VM与基于kernel支持的容器（如cgroup和namespace）一样重要。我们可以说PouchContainer的容器引擎部分非常纯净。对于容器编排的更多责任则依赖于高级编排技术，如 [Kubernetes](https://github.com/kubernetes/kubernetes), [Mesos](https://github.com/apache/mesos)。

## PouchContainer与Docker有什么不同

PouchContainer和Docker对于用户来说都是优秀的容器解决方案。乍一看，它们都在做类似的事情。但更具体地说，他们在每个人的目标上强调的东西有所不同。 PouchContainer更加重视应用程序的体验，而Docker则主张“一个进程一个容器”。 在某些特殊场景中，PouchContainer确实不能忽视容器技术带来的隔离威胁，而Docker则严重依赖kernel来实现隔离。 另一方面，PouchContainer为周围的生态带来了开放的态度，虽然docker也在这方面也有建树，但也许并不是那么多。

在这里，我们列出了PouchContainer的一些附加功能：

* 富容器：这意味着容器中不再仅有一个应用进程。每个容器都有自己的初始化进程，和可根据用户的需要在内部进行部署的其他系统服务。
* 强大的隔离性：PouchContainer可以通过[runV](https://github.com/hyperhq/runv)和[clearcontainer](https://github.com/clearcontainers/runtime)创建具有hypervisor技术的VM。
* 高内核兼容性：PouchContainer支持非常广泛的kernel版本。业界将内核版本升级到3.10+仍然是一条非常漫长的道路。 PouchContainer可以让老版本kernel的也一起享受新的容器技术。
* P2P镜像分发：在一个非常大的数据中心，镜像分发会给网络带来很大的负荷。 PouchContainer利用P2P镜像分发解决方案来改善这一点。

## PounchContainer和Kubernetes有什么不同

Kubernetes是一个开源项目，用于管理跨多机容器化的应用程序。它为应用程序的部署，维护和扩展提供基本机制。而PouchContainer则主要关注具有富容器运行多样性的容器管理。如果想要更清楚地了解PouchContainer和Kubernetes之间的关系，您可以参考[生态系统架构](docs/architecture.md#ecosystem-architecture)。

如果您正在为超过100个节点的集群场景选择整体的容器解决方案，并且想要应用的维护变得友好且省力，您可以选择Kubernetes。如果您希望强化容器的运行，使其能高效地为应用打包，并用相同的操作来标准化您的异构设施，那么PouchContainer就是您想要的。

更重要的是，在数据中心的架构中，Kubernetes和PouchContainer位于不同的层。我们可以说Kubernetes位于PouchContainer的上层。实际上，我们可以有效地结合Kubernetes和PouchContainer。 PouchContainer在runtime solution中发挥作用，而Kubernetes则在编排过程中发挥作用。PouchContainer从基础设施中接管细粒度资源，如CPU，内存，网络，磁盘等。然后，它为上层Kubernetes提供这些资源指标用于调度。当Kubernetes为了满足应用程序的维护需求而运行时，它可以将请求传递给PouchContainer，以便为应用程序提供安全且隔离的容器载体。

## 富容器是什么

当容器化应用程序时，富容器是一种非常有用的容器模式。此模式可帮助技术人员轻松完成庞大的应用包。它提供了有效的方法来安装更多的基本软件或系统服务，除了在单个容器中的目标应用程序。然后，容器中的应用可以像平常一样在虚拟机或物理机中运行。这是一种功能以应用程序为中心的通用模式。这种模式对开发人员和运营人员来说都很友好。特别是对运营人员来说，他们可以像往常一样使用他们可能需要的所有必要工具或服务流程来维护容器中的应用程序。
富容器不是PouchContainer提供的默认模式，这是PouchContainer为扩展用户的容器体验而产生的额外模式。用户仍然可以通过关闭富容器来管理普通容器。有关详细信息，请参阅[富容器](./docs/features/pouch_with_rich_container.md).
。

## containerd和PouchContainer之间的关系是什么

敬请期待

## PouchContainer使用的镜像存储驱动是什么

PouchContainer使用overlay2作为其默认图像存储的驱动程序。更具体的说，containerd 1.0.0+开始使用overlay2来存储图像。
虽然目前containerd是pouchContainer中的基本组件。对于Linux的主线内核，只有内核版本为4.0+支持overlay2。因此从理论上讲，PouchContainer只能在基于内核4.0+的linux发行版上运行。但是，RedHat系列，如CentOS7.2+等，自内核版本3.10.327起，稳定支持overlay2。有关RedHat的更多信息，请参阅[红帽企业级Linux发布日期](https://access.redhat.com/articles/3078#RHEL7)。

## PouchContainer支持Kubernetes吗

是的。PouchContainer已经实现了所有CRI接口(container runtime interface)。您可以在Kubernetes下层使用PouchContainer。有关详细信息，请参阅[Kubernetes](./docs/kubernetes)。

## PouchContainer支持Mesos/DCOS吗
敬请期待。

## PouchContainer的版本规则是什么

我们在[SemVer](http://semver.org/)的基础上制定了PouchContainer的版本规则。简言之，我们遵循的MAJOR.MINOR.PATCH版本号，如：0.2.1, 1.1.3。有关详情信息，请参阅[SemVer](http://semver.org/)。

## PouchContainer的规划蓝图
请见[ROADMAP.md](./ROADMAP.md)

## 如何为PouchContainer贡献代码

我们十分欢迎您能对PouchContainer贡献自己的力量。

更多详情信息，请参阅[CONTRIBUTION.md](./CONTRIBUTING.md)





