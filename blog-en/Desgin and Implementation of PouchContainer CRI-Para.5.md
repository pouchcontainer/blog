
Because all the containers in the Pod share the Network Namespace, we only need to configure the Network Namespace when creating the infra container.


The network functions of containers in the Kubernetes ecosystem are implemented by CNI. Similar to CRI, CNI is also a standard interface, and various network solutions can seamlessly access Kubernetes as long as the interface is implemented. The CNI Manager in CRI Manager is a simple package for CNI. It will load the configuration file in the directory /etc/cni/net.d during the initialization process, as shown below:
```
$ cat >/etc/cni/net.d/10-mynet.conflist <<EOF
{
        "cniVersion": "0.3.0",
        "name": "mynet",
        "plugins": [
          {
                "type": "bridge",
                "bridge": "cni0",
                "isGateway": true,
                "ipMasq": true,
                "ipam": {
                        "type": "host-local",
                        "subnet": "10.22.0.0/16",
                        "routes": [
                                { "dst": "0.0.0.0/0" }
                        ]
                }
          }
        ]
}
EOF
```

It specifies the CNI plugin that will be used to configure the Pod network, such as the bridge above, and some network configuration information, such as the subnet range and routing configuration of the Pod of the node.

Next, we will show how to add a Pod to the CNI network through specific steps:

1. When the container manager is called to create an infra container, set NetworkMode to "None" mode, which means creating a unique Network Namespace for the infra container without any configuration.
2. According to the PID corresponding to the infra container, obtain the corresponding Network Namespace path /proc/{pid}/ns/net.
3. Call the SetUpPodNetwork method of CNI Manager. The core parameter is the Network Namespace path obtained in step 2. The work of this method is to call the CNI plugin specified during the initialization of CNI Manager, such as the bridge above. Configure the Network Namespace specified in the parameter, including creating various network devices, performing various network configurations, and adding the Network Namespace to the CNI network corresponding to the plugin.

For most Pods, the network configuration is in accordance with the above steps, most of the work will be done by CNI and the corresponding CNI plugin for us. But for some special Pods, they will set their own network mode to "Host", which is to share the Network Namespace with the host. At this time, we only need to set the NetworkMode to "Host" when calling the Container Manager to create the infra container, and skip the configuration of the CNI Manager.

For other containers in the Pod, whether the Pod is in the "Host" network mode or has a separate Network Namespace, you only need to configure the NetworkMode to "Container" mode when you call Container Manager to create a container, and join the Network Namespace where the infra container is located.

[origin doc link](https://github.com/pouchcontainer/blog/blob/master/blog-cn/PouchContainer%20CRI%E7%9A%84%E8%AE%BE%E8%AE%A1%E4%B8%8E%E5%AE%9E%E7%8E%B0.md)


