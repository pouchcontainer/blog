### 2. Overview of CRI design



![cri-1.png | left | 827x299](https://cdn.yuque.com/lark/0/2018/png/103564/1527478355304-a20865ae-81b8-4f13-910d-39c9db4c72e2.png "")


As shown in the figure above, Kubelet is the Node Agent of the Kubernetes cluster, which will supervise the state of containers to ensure they all run as expected. To achieve this, Kubelet calls corresponding CRI interface to synchronize the container.

CRI shim works as an interface conversion layer, which converts the CRI interface to that corresponding to the underlying container runtime，and calls interface and returns result. For some container runtimes, CRI shim works as an independent process. For example, in the case that Docker is the container runtime of Kubernetes, Kubelet initializes with a Docker shim process which is Docker's CRI shime. For PouchContainer, CRI shim is embedded in Pouchd, which we call CRI manager. We will discuss this in more detail in the next section when we discuss the architecture of PouchContainer.

CRI is essentially gRPC interface. Kubelet has an embedded gRPC Client, while CRI shim has an  embedded gRPC Server. Every time Kubelet calls CRI interface, it will be converted to a gRPC request and then sent to gRPC Server in CRI shim by gRPC Client. Server processes the request and returns a result by calling underlying container runtimes,  which carries out the whole precess of calling CRI interface.

gRPC interface defined by CRI can be categorized into two types, ImageService and RuntimeService: ImageService is responsible for managing images of container, while RuntimeService is responsible for managing container’s life-cycle and interacting with container (exec/attach/port-forward).

### 3. CRI Manager Architectural design



![yzz's pic.jpg | left | 827x512](https://cdn.yuque.com/lark/0/2018/jpeg/95844/1527582870490-a9b9591d-d529-4b7d-bc5f-69514ef115e7.jpeg "")


In the whole structure of PouchContainer, CRI Manager implements all CRI interfaces and serves as CRI shim in PouchContainer. When kubelet calls some CRI interface, request would go through Kubelet’s gPRC client and eventually sent to gRPC Server, as shown in the graph. Server will analyze the request and call CRI Manager to handle the request with appropriate method. 

We will now exemplify functionalities of every module of CRI Manager. When requested to create a new Pod, CRI Manager will convert the CRI formatted configuration obtained from the request to a format that satisfies requirements of PouchContainer, call Image Manager and Container Manager to respectively get necessary images and to create new container. CRI Manager also calls CNI Manager to config network of the Pod with CNI plugins. At last, Stream Server will execute interact request such as exec/attach/portforward.

To note that CNI Manager and Stream Server are submodules of CRI Manager. CRI Manager, Container Manager and Image Manager are at the same level, all contained in a binary file — Pouchd. Therefore, they only interact with functions. Unlike interaction between Docker shim and Docker, no cost of remote calls is required for the interaction of these three modules. We will go deep inside CRI Manager and explore implementations of its core functionalities in the following passage.