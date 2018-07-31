# 带插件的PouchContainer

为了让用户运行自定义代码，`golang 1.8`支持引入插件框架。在插件框架中，我们支持用户在以下阶段添加自定义代码：

- 预启动守护阶段
- 预停止守护阶段
- 预创建容器阶段
- 预启动容器阶段
- 预创建端点容器阶段

以上5个阶段点由`DaemonPlugin`和`ContainerPlugin`组织，两个插件定义如下：

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

这两个插件符号将通过名称`DaemonPlugin`和`ContainerPlugin`从共享对象文件中获取，如下所示：

```
p, _ := plugin.Open("path_to_shared_object_file")
daemonPlugin, _ := p.Lookup("DaemonPlugin")
containerPlugin, _ := p.Lookup("ContainerPlugin")
```

## 范例

定义两个在对应阶段打印日志的插件符号：

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

用如下命令行生成：

```
go build -buildmode=plugin -ldflags "-pluginpath=plugins_$(date +%s)" -o hook_plugin.so
```

要使用生成的共享对象文件，启动`--plugin = path_to_hook_plugin.so` flag 的 pouchd，然后当你启动停止守护进程并创建容器时，在日志中会有如下日志：

```
pre-start hook in daemon is called
pre create method called
pre-stop hook in daemon is called
```

当你启动一个容器时，config.json文件（其地址为$ home_dir / containerd / state / io.containerd.runtime.v1.linux / default / $ container_id / config.json）将包含以上代码中定义的预启动挂钩，例如：

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

如果你使用如上代码，每当你启动容器，路径`/tmp/pre_start_hook`下的文件将被调用。

## 使用

- 在预启动守护程序阶段，您可以启动协助过程，如网络插件和dfget代理，其需要pouchd，并且其生命周期与pouchd相同。
- 在预停止守护程序阶段，你可以优雅地停止辅助进程，但这并不是实时有效，因为可能会被SIGKILL杀死。
- 在容器的预创建阶段，你可以根据一些规则改变输入流，在一些公司里面他们会有过时的编排系统，其用环境变量来传递一些限制条件，这些环境变量是pouchContainer中的属性。你可以在预创建阶段将环境变量的值转换成PouchContainer create Api中用到的ContainerConfig或者HostConfig的属性。
- 在容器的预开始阶段，你可以设置更多针对oci规格的钩子，你可以在入口启动开始前做一些特殊的事情，也可以针对钩子运行的顺序设置优先级。libnetwork的优先级是0，所以如果你期望它在容器启动阶段时先于network运行，你就该给它设一个大于0的优先级，反之亦然。
- 在容器的端点预创建阶段，你可以返回端点的优先级和该端点是否需要激活分解器以及该端点的通用参数。