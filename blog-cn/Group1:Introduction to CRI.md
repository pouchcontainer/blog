### 1. Introduction to CRI
At the bottom level of each Kubernetes node there is a program dedicated to containers creation and deletion, Kubernetes will call its interface to complete the container scheduling. We call this layer of software the Container Runtime, and the famous Docker is the representative.

Of course, there is not only Docker developed for this purpose, including CoreOS rkt, hyper.sh runV, Google's gvisor, and the protagonist PouchContainer, they all provide complete container operations that can be used to create containers with different characteristics. Different containers have their own unique advantages to meet the needs of different users, so it is imperative that Kubernetes supports the running of multiple containers

Originally, Kubernetes natively built an interface to Docker, and the community then integrated the rkt interface in Kubernetes 1.3, making it an optional container runtime other than Docker. However, at this time, both the Docker and the rkt call are strongly coupled with the core code of Kubernetes, which will undoubtedly bring the following two problems:

1. The emerging container, such as a rising star like PouchContainer, is very difficult to join Kubernetes. The developer of the container runtime must have a very deep understanding of the Kubernetes code (at least Kubelet) in order to successfully complete the docking between the two.

2. Kubernetes code will be more difficult to maintain, which is also reflected in two aspects: (1) the interface of various container runtimes is hard-coded into Kubernetes, which will make Kubernetes' core code become bloated, and (2) any subtle changes to the runtime interface will cause changes to the Kubernetes core code, which will increase the instability of Kubernetes

In order to solve these problems, the community introduced CRI (Container Runtime Interface) in Kubernetes 1.5. By defining a set of container runtime public interfaces, Kubernetes shields the call interfaces of various container runtimes from the core code. Kubernetes core code only opens to the interface from the abstract layer. For various container runtimes, Kubernetes can be successfully accessed as long as the various interfaces defined in the CRI are met, making it a container runtime option. The solution is simple, but it is a liberation for Kubernetes community maintainers and container runtime developers.




