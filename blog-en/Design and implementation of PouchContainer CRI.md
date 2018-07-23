## 2.CRI Design Overview

![cri-1.png | left | 827x299](https://cdn.yuque.com/lark/0/2018/png/103564/1527478355304-a20865ae-81b8-4f13-910d-39c9db4c72e2.png "")

As shown in the figure above, the Kubelet on the left is the node agent of the Kubernetes cluster, which monitors the state of the containers on this node to ensure that they all run as expected. In order to achieve this goal, Kubelet will continuously call the relevant CRI interface to synchronize the container.

CRI shim can be regarded as an interface translation layer which can convert CRI interface to the interface of corresponding underlying container at runtime and call this interface, as well as get the response. CRI shim exists as a unique process for some running containers. For instance, when Docker is selected as the container for Kubernetes, Kubelet will start with a Docker shim process, which is Docker's CRI shime. For PouchContainer, its CRI shim is embedded in Pouchd, which is called CRI manager. In this regard, we will give more details when talking about PouchContainer related architecture in the next section.

CRI is essentially a set of gRPC interfaces. Kubelet has a built-in gRPC Client, and CRI shim has a built-in gRPC Server. Each time the call from Kubelet to the CRI interface will be converted to a gRPC request sent by the gRPC Client to the gRPC Server in the CRI shim. Server calls the underlying container to process the request and return the result, thus completing a CRI interface call.

The CRP-defined gRPC interface can be divided into two categories, ImageService and RuntimeService: where ImageService is responsible for managing the container's image, while RuntimeService is responsible for managing the container's lifecycle and interacting with the container (exec/attach/port-forward).

## 3.CRI Manager architecture design

![yzz's pic.jpg | left | 827x512](https://cdn.yuque.com/lark/0/2018/jpeg/95844/1527582870490-a9b9591d-d529-4b7d-bc5f-69514ef115e7.jpeg "")

In the entire architecture of PouchContainer, CRI Manager implements all the interfaces defined by CRI and plays the role of CRI shim in PouchContainer. When Kubelet calls a CRI interface, the request is sent to the gRPC Server in the above figure via the Kublet's gRPC Client. Server will parse the request and call the corresponding method of CRI Manager to process.

Let's take a look at an example to get a brief look at the founctionality of each module. For example, when the arrived request is to create a Pod, the CRI Manager will first convert the acquired CRI format configuration into a format that meets the requirements of the PouchContainer interface, call Image Manager to pull the required image, and then call Container Manager to create the required container, and call CNI Manager, using the CNI plugin to configure the Pod network. Finally, Stream Server handles interactive type CRI requests, such as exec/attach/portforward.

It is worth noting that CNI Manager and Stream Server are sub-modules of CRI Manager, and CRI Manager, Container Manager and Image Manager are three equal modules, all located in the same binary file Pouchd, so the calls between them are the most straightforward function call, and there is no remote call overhead required like when Docker shim interacts with Docker. Next, we will enter the CRI Manager to gain a deeper understanding of the implementation of important functions.


