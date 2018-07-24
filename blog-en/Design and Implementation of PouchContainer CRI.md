### 5. Pod Network Configuration

Since all containers inside Pod share a Network Namespace, we only need to configure Network Namespace when creating the infra container.

All network functions of containers in Kubernetes ecosystem are implemented via CNI. Similar to CRI, CNI is a set of standard interfaces and any network schema implementing these interfaces can be integrated into Kubernetes seamlessly. CNI Manager in CRI Manager is a simple encapsulation of CNI. It will load configuration files under `/etc/cni/net.d` during initialization, as follows:

```sh
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

In which the CNI plugins used to configure Pod network are specified, such as the `bridge` above, as well as some network configurations, like the subnet range and routing configuration of Pod on current node.

Now let's show how to register a Pod into CNI step by step:

1. When creating a infra container via container manager, set `NetworkMode` as "None", indicating creating a unique Network Namespace owned by this infra container without any configuration.
2. Based on the corresponding PID of infra container, the directory of corresponding Network Namespace (`/proc/{pid}/ns/net`) can be obtained.
3. Invoke `SetUpPodNetwork` method of CNI Manager, the core argument is the Network Namespace directory obtained in step 2. This method in turn invokes CNI plugins specified during the initialization of CNI Manager, such as 'bridge' in above snippet, configuring Network Namespace specified in argumants, including creating various network devices, configuring network, registering this Network Namespace into corresponding CNI network.

For most Pods, network configuration can be done through the above steps and most of the work are done by CNI and corresponding plugins. But for some special Pods, whose network mode are set to "Host", that is, sharing Network Namespace with host machine. In this case, we only need to set `NetworkMode` to "Host" when creating infra container via Container Manager, and skip the configuration of CNI Manager.

For other containers in a Pod, either the Pod is in "Host" mode, or it has independent Network Namespace, we only need to set `NetworkMode` to "Container" when creating containers via Container Manager, and join the Network Namespace that infra container belongs to.
