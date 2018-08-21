# PouchContainer with plugin

为了运行在某些点引起的由用户提供的自定义代码，我们提供了了一个介绍golang 1.8的插入框架。此时在插入框架中我们能够让用户在文件点中加入自定义代码。


* 守护进程开始之前
* 守护进程结束前
* 容器创建前
* 容器开始前
* 容器创建结束前

以上四点是由两个插入界面提供的，分别是DaemonPlugin和ContainerPlugin，定义如下：


```
// DaemonPlugin defines in which place does pouchd support plugin
type DaemonPlugin interface {
    // PreStartHook is invoked by pouchd before real start, in this hook user could start dfget proxy or other
    // standalone process plugins
    PreStartHook() error

    // PreStopHook is invoked by pouchd before daemon process exit, not a promise if daemon is killed, in this
    // hook user could stop the process or plugin started by PreStartHook
    PreStopHook() error
}

// ContainerPlugin defines places where a plugin will be triggered in container lifecycle
type ContainerPlugin interface {
  // PreCreate defines plugin point where receives an container create request, in this plugin point user
  // could change the container create body passed-in by http request body
  PreCreate(io.ReadCloser) (io.ReadCloser, error)

  // PreStart returns an array of priority and args which will pass to runc, the every priority
  // used to sort the pre start array that pass to runc, network plugin hook always has priority value 0.
  PreStart(interface{}) ([]int, [][]string, error)

  //NetworkGenericParams accepts the container id and env of this container and returns the priority of this endpoint
  // and if this endpoint should enable resolver and a map which will be used as generic params to create endpoints of
  // this container
  PreCreateEndpoint(string, []string) (priority int, disableResolver bool, genericParam map[string]interface{})
}

```

这两个插入符号会被记为分享的目标文件中的‘DaemonPlugin’(后台运行插件)和‘ContainerPlugin’(容器插件），示例如下：

```
p, _ := plugin.Open("path_to_shared_object_file")
daemonPlugin, _ := p.Lookup("DaemonPlugin")
containerPlugin, _ := p.Lookup("ContainerPlugin")
```

## example

定义两个只在对应点处打印某些日志在的插入符号

```
package main

import (
    "fmt"
    "io"
)

var ContainerPlugin ContPlugin

type ContPlugin int

var DaemonPlugin DPlugin

type DPlugin int

func (d DPlugin) PreStartHook() error {
    fmt.Println("pre-start hook in daemon is called")
    return nil
}

func (d DPlugin) PreStopHook() error {
    fmt.Println("pre-stop hook in daemon is called")
    return nil
}

func (c ContPlugin) PreCreate(in io.ReadCloser) (io.ReadCloser, error) {
    fmt.Println("pre create method called")
    return in, nil
}

func (c ContPlugin) PreStart(interface{}) ([]int, [][]string, error) {
    fmt.Println("pre start method called")
    // make this pre-start hook run after network in container setup
    return []int{-4}, [][]string{{"/usr/bin/touch", "touch", "/tmp/pre_start_hook"}}, nil
}

func (c ContPlugin) PreCreateEndpoint(string, []string) (priority int, disableResolver bool, genericParam map[string]interface{}) {
    fmt.Println("pre create endpoint")
    return
}

func main() {
    fmt.Println(ContainerPlugin, DaemonPlugin)
}
```

然后建立某些命令行，如下所示：

```
go build -buildmode=plugin -ldflags "-pluginpath=plugins_$(date +%s)" -o hook_plugin.so
```

利用生成的分享目标文件，开始封装其中带有旗标的`--plugin=path_to_hook_plugin.so`，然后当你开始结束守护进程和创建容器的时候，在某些日志中会像如下这样：


```
pre-start hook in daemon is called
pre create method called
pre-stop hook in daemon is called
```

当你开始一个容器时，config.json文件（位置在$home_dir/containerd/state/io.containerd.runtime.v1.linux/default/$container_id/config.json）会包含预开始外挂脚本，特别在上述代码中，例如：

```
    "hooks": {
        "prestart": [
            {
                "args": [
                    "libnetwork-setkey",
                    "f67df14e96fa4b94a6e386d0795bdd2703ca7b01713d48c9567203a37b05ae3d",
                    "8e3d8db7f72a66edee99d4db6ab911f8d618af057485731e9acf24b3668e25b6"
                ],
                "path": "/usr/local/bin/pouchd"
            },
            {
                "args": [
                    "touch",
                    "/tmp/pre_start_hook"
                ],
                "path": "/usr/bin/touch"
            }
        ]
    }
```

如果你准确的运用上诉代码，每一次你打开一个容器，在/tmp/pre_start_hook的文件会被触达。


## usage

* 在守护进程开始之前，你可以开始帮助进程，比如被pouchd以及生命周期跟pouchd一样的进程所需要的帮助进程，比如网络插件和网络代理。

* 在守护进程结束前，你可以优雅地结束帮助进程，但是这个事件并不一定会被触发，因为pouchd可能会被SIGKILL信号终止。

* 在容器创建前，你可以根据某些规则改变输入流，在某些公司他们会有一些编配编排系统，这些系统利用env去传递一些Pouch容器中的限制，然后你可以利用这些事件去转换env中的值转换到PouchContainer创建api中的ContainerConfig或HostConfig的属性。

* 在容器开始之前，你可以创建更多预开始钩子用于oci容器，其中在容器入口开始之前，你可以做一些具体的事情。优先级决定钩子的执行顺序。libnetwork钩子优先级为0，所以在容器设置中，如果钩子被期望在网络的容器之前设置，你需要设置优先级为一个大于0的数，反之亦然。

* 在容器创建停止之前，你可以返回这个停止点的优先级，如果这个停止点需要解析器和这个停止点的通用参数。
