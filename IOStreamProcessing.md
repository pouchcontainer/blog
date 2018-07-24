### 6. IO stream processing

Kubernetes provides features such as `kubectl exec/attach/port-forward` to enable direct interaction between the user and a specific Pod or container. The command are as follows:

```sh
aster $ kubectl exec -it shell-demo -- /bin/bash
root@shell-demo:/# ls
bin   dev  home  lib64  mnt  proc  run   srv  tmp  var
boot  etc  lib   media  opt  root  sbin  sys  usr
root@shell-demo:/#
```
As we can see, `exec` a Pod is equivalent to `ssh` logging into the container. Below, we analyze the processing of IO requests in Kubernetes in the execution flow of `kubectl exec` and the role that CRI Manager plays.
 
![stream-3.png | left | 827x296](https://cdn.yuque.com/lark/0/2018/png/103564/1527478375654-1c891ac5-7dd0-4432-9f72-56c4feb35ac6.png "")


As shown above, the steps to execute a kubectl exec command are as follows:

1.The essence of the `kubectl exec` command is to execute an exec command on a container in the Kubernetes cluster and forward the resulting IO stream to the users. So the request will first be forwarded to the Kubelet of the node where the container is located, and the Kubelet will then call the `Exec` interface in the CRI according to the configuration. The requested configuration parameters are as follows:

```go
type ExecRequest struct {
	ContainerId string	// execute the aimed container of exec
	Cmd []string	// specific execution of the exec command
	Tty bool	// whether to execute the exec command in a TTY
	Stdin bool	// whether to include Stdin flow
	Stdout bool	// whether to include Stdout flow
	Stderr bool	// whether to include Stderr flow
}
```

2.Surprisingly, CRI Manager's `Exec` method does not directly call Container Manager, and executes the exec command on the target container, but instead calls the built-in Stream Server `GetExec` method.

3.The work done by the Stream Server `GetExec` method is to save the contents of the exec request to the Request Cache shown in the figure above, and return a token. With this token, we can retrieve the corresponding exec request from the Request Cache. Finally, the token is written to a URL and returned to the ApiServer as a result layer.

4.ApiServer uses the returned URL to directly initiate a http request to the Stream Server of the node where the target container is located. The request header contains the "Upgrade" field, which is required to upgrade the http protocol to a streaming protocol such as websocket or SPDY to support multiple IOs. Stream processing, this article we take SPDY as an example.

5.The Stream Server processes the request sent by the ApiServer. First, the previously saved exec request configuration is obtained from the Request Cache according to the token in the URL. After that, reply to the http request, agree to upgrade the protocol to SPDY, and wait for the ApiServer to create specified numbers of streams according to the configuration of the exec request, corresponding to the standard input Stdin, the standard output Stdout, and the standard error output Stderr.

6.After the Stream Server obtains the specified number of Streams, the Container Manager's `CreateExec` and `startExec` methods are invoked in turn to perform an exec operation on the target container and forward the IO stream to the corresponding stream.

7.Finally, the ApiServer forwards the data of each stream to the user, and starts the IO interaction between the user and the target container.

In fact, before the introduction of CRI, Kubernetes handled the IO in the same way as we expected. Kubelet will execute the exec command directly on the target container and forward the IO stream back to the ApiServer. But this will put Kubelet under too much pressure, and all IO streams need to be forwarded through it, which is obviously unnecessary. Therefore, although the above processing is complicated at first glance, it effectively alleviates the pressure of Kubelet and makes the processing of IO more efficient.
