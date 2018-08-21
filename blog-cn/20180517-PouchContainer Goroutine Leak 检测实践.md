## <a name="pobkqe"></a>0. 引言

[PouchContainer](https://github.com/alibaba/pouch) 是阿里巴巴集团开源的一款容器运行时产品，它具备强隔离和可移植性等特点，可用来帮助企业快速实现存量业务容器化，以及提高企业内部物理资源的利用率。

PouchContainer 同时还是一款 golang 项目。在此项目中，大量运用了 goroutine 来实现容器管理、镜像管理和日志管理等模块。goroutine 是 golang 在语言层面就支持的用户态 “线程”，这种原生支持并发的特性能够帮助开发者快速构建高并发的服务。

虽然 goroutine 容易完成并发或者并行的操作，但如果出现 channel 接收端长时间阻塞却无法唤醒的状态，那么将会出现 __goroutine leak__ 。 goroutine leak 同内存泄漏一样可怕，这样的 goroutine 会不断地吞噬资源，导致系统运行变慢，甚至是崩溃。为了让系统能健康运转，需要开发者保证 goroutine 不会出现泄漏的情况。 接下来本文将从什么是 goroutine leak, 如何检测以及常用的分析工具来介绍 PouchContainer 在 goroutine leak 方面的检测实践。

## <a name="toxmge"></a>1. Goroutine Leak

在 golang 的世界里，你能支配的土拨鼠有很多，它们既可以同时处理一大波同样的问题，也可以协作处理同一件事，只要你指挥得当，问题就能很快地处理完毕。没错，土拨鼠就是我们常说的 `goroutine` ，你只要轻松地 `go` 一下，你就拥有了一只土拨鼠，它便会执行你所指定的任务：

```go
func main() {
	waitCh := make(chan struct{})
	go func() {
		fmt.Println("Hi, Pouch. I'm new gopher!")
		waitCh <- struct{}{}
	}()

	<-waitCh
}
```

正常情况下，一只土拨鼠完成任务之后，它将会回笼，然后等待你的下一次召唤。但是也有可能出现这只土拨鼠很长时间没有回笼的情况。

```go
func main() {
        // /exec?cmd=xx&args=yy runs the shell command in the host
        http.HandleFunc("/exec", func(w http.ResponseWriter, r *http.Request) {
                defer func() { log.Printf("finish %v\n", r.URL) }()
                out, err := genCmd(r).CombinedOutput()
                if err != nil {
                        w.WriteHeader(500)
                        w.Write([]byte(err.Error()))
                        return
                }
                w.Write(out)
        })
        log.Fatal(http.ListenAndServe(":8080", nil))
}

func genCmd(r *http.Request) (cmd *exec.Cmd) {
        var args []string
        if got := r.FormValue("args"); got != "" {
                args = strings.Split(got, " ")
        }

        if c := r.FormValue("cmd"); len(args) == 0 {
                cmd = exec.Command(c)
        } else {
                cmd = exec.Command(c, args...)
        }
        return
}
```

上面这段代码会启动 HTTP Server，它将允许客户端通过 HTTP 请求的方式来远程执行 shell 命令，比如可以使用 `curl "{ip}:8080/exec?cmd=ps&args=-ef"` 来查看 Server 端的进程情况。执行完毕之后，土拨鼠会打印日志，并说明该指令已执行完毕。

但是有些时候，请求需要土拨鼠花很长的时间处理，而请求者却没有等待的耐心，比如 `curl -m 3 "{ip}:8080/exec?cmd=dosomething"`，即在 3 秒内执行完某一条命令，不然请求者将会断开链接。由于上述代码并没有检测链接断开的功能，如果请求者不耐心等待命令完成而是中途断开链接，那么这个土拨鼠也只有在执行完毕后才会回笼。可怕的是，遇到这种 `curl -m 1 "{ip}:8080/exec?cmd=sleep&args=10000"` ，没法及时回笼的土拨鼠会占用系统的资源。

这些流离在外、不受控制的土拨鼠，就是我们常说的 __goroutine leak__ 。造成 goroutine leak 的原因有很多，比如 channel 没有发送者。运行下面的代码之后，你会发现 runtime 会稳定地显示目前共有 2 个 goroutine，其中一个是 `main` 函数自己，另外一个就是一直在等待数据的土拨鼠。

```go
func main() {
        logGoNum()

        // without sender and blocking....
        var ch chan int
        go func(ch chan int) {
                <-ch
        }(ch)

        for range time.Tick(2 * time.Second) {
                logGoNum()
        }
}

func logGoNum() {
        log.Printf("goroutine number: %d\n", runtime.NumGoroutine())
}
```

造成 goroutine leak 有很多种不同的场景，本文接下来会通过描述 Pouch Logs API 场景，介绍如何对 goroutine leak 进行检测并给出相应的解决方案。

## <a name="eav8ss"></a>2. Pouch Logs API 实践
### <a name="8rtcwv"></a>2.1 具体场景

为了更好地说明问题，本文将 Pouch Logs HTTP Handler 的代码进行简化：

```go
func logsContainer(ctx context.Context, w http.ResponseWriter, r *http.Request) {
    ...
    writeLogStream(ctx, w, msgCh)
    return
}

func writeLogStream(ctx context.Context, w http.ResponseWriter, msgCh <-chan Message) {
    for {
        select {
        case <-ctx.Done():
            return
        case msg, ok := <-msgCh:
            if !ok {
                return
            }
            w.Write(msg.Byte())
        }
    }
}
```

Logs API Handler 会启动 goroutine 去读取日志，并通过 channel 的方式将数据传递给 `writeLogStream` ，`writeLogStream` 便会将数据返回给调用者。这个 Logs API 具有 __跟随__ 功能，它将会持续地显示新的日志内容，直到容器停止。但是对于调用者而言，它随时都会终止请求。那么我们怎么检测是否存在遗留的 goroutine 呢？

> 当链接断开之后，Handler 还想给 Client 发送数据，那么将会出现 write: broken pipe 的错误，通常情况下 goroutine 会退出。但是如果 Handler 还在长时间等待数据的话，那么就是一次 goroutine leak 事件。

### <a name="6x6rvw"></a>2.2 如何检测 goroutine leak？

对于 HTTP Server 而言，我们通常会通过引入包 `net/http/pprof` 来查看当前进程运行的状态，其中有一项就是查看 goroutine stack 的信息，`{ip}:{port}/debug/pprof/goroutine?debug=2`  。我们来看看调用者主动断开链接之后的 goroutine stack 信息。

```powershell
# step 1: create background job
pouch run -d busybox sh -c "while true; do sleep 1; done"

# step 2: follow the log and stop it after 3 seconds
curl -m 3 {ip}:{port}/v1.24/containers/{container_id}/logs?stdout=1&follow=1

# step 3: after 3 seconds, dump the stack info
curl -s "{ip}:{port}/debug/pprof/goroutine?debug=2" | grep -A 10 logsContainer
github.com/alibaba/pouch/apis/server.(*Server).logsContainer(0xc420330b80, 0x251b3e0, 0xc420d93240, 0x251a1e0, 0xc420432c40, 0xc4203f7a00, 0x3, 0x3)
        /tmp/pouchbuild/src/github.com/alibaba/pouch/apis/server/container_bridge.go:339 +0x347
github.com/alibaba/pouch/apis/server.(*Server).(github.com/alibaba/pouch/apis/server.logsContainer)-fm(0x251b3e0, 0xc420d93240, 0x251a1e0, 0xc420432c40, 0xc4203f7a00, 0x3, 0x3)
        /tmp/pouchbuild/src/github.com/alibaba/pouch/apis/server/router.go:53 +0x5c
github.com/alibaba/pouch/apis/server.withCancelHandler.func1(0x251b3e0, 0xc420d93240, 0x251a1e0, 0xc420432c40, 0xc4203f7a00, 0xc4203f7a00, 0xc42091dad0)
        /tmp/pouchbuild/src/github.com/alibaba/pouch/apis/server/router.go:114 +0x57
github.com/alibaba/pouch/apis/server.filter.func1(0x251a1e0, 0xc420432c40, 0xc4203f7a00)
        /tmp/pouchbuild/src/github.com/alibaba/pouch/apis/server/router.go:181 +0x327
net/http.HandlerFunc.ServeHTTP(0xc420a84090, 0x251a1e0, 0xc420432c40, 0xc4203f7a00)
        /usr/local/go/src/net/http/server.go:1918 +0x44
github.com/alibaba/pouch/vendor/github.com/gorilla/mux.(*Router).ServeHTTP(0xc4209fad20, 0x251a1e0, 0xc420432c40, 0xc4203f7a00)
        /tmp/pouchbuild/src/github.com/alibaba/pouch/vendor/github.com/gorilla/mux/mux.go:133 +0xed
net/http.serverHandler.ServeHTTP(0xc420a18d00, 0x251a1e0, 0xc420432c40, 0xc4203f7800)
```

我们会发现当前进程中还存留着 `logsContainer` goroutine。因为这个容器没有输出任何日志的机会，所以这个 goroutine 没办法通过 `write: broken pipe` 的错误退出，它会一直占用着系统资源。那我们该怎么解决这个问题呢？

### <a name="vhqywi"></a>2.3 怎么解决？

golang 提供的包 `net/http`  有监控链接断开的功能:

```go
// HTTP Handler Interceptors
func withCancelHandler(h handler) handler {
        return func(ctx context.Context, rw http.ResponseWriter, req *http.Request) error {
                // https://golang.org/pkg/net/http/#CloseNotifier
                if notifier, ok := rw.(http.CloseNotifier); ok {
                        var cancel context.CancelFunc
                        ctx, cancel = context.WithCancel(ctx)

                        waitCh := make(chan struct{})
                        defer close(waitCh)

                        closeNotify := notifier.CloseNotify()
                        go func() {
                                select {
                                case <-closeNotify:
                                        cancel()
                                case <-waitCh:
                                }
                        }()
                }
                return h(ctx, rw, req)
        }
}
```

当请求还没执行完毕时，客户端主动退出了，那么 `CloseNotify()` 将会收到相应的消息，并通过 `context.Context` 来取消，这样我们就可以很好地处理 goroutine leak 的问题了。在 golang 的世界里，你会经常看到 __读__ 和 __写__ 的 goroutine，它们这种函数的第一个参数一般会带有 `context.Context` , 这样就可以通过 `WithTimeout` 和 `WithCancel` 来控制 goroutine 的回收，避免出现泄漏的情况。

> CloseNotify 并不适用于 Hijack 链接的场景，因为 Hijack 之后，有关于链接的所有处理都交给了实际的 Handler，HTTP Server 已经放弃了数据的管理权。

那么这样的检测可以做成自动化吗？下面会结合常用的分析工具来进行说明。

## <a name="shc8xg"></a>3. 常用的分析工具

### <a name="bshgpi"></a>3.1 net/http/pprof 

在开发 HTTP Server 的时候，我们可以引入包 `net/http/pprof` 来打开 debug 模式，然后通过 `/debug/pprof/goroutine` 来访问 goroutine stack 信息。一般情况下，goroutine stack 会具有以下样式。

```plain
goroutine 93 [chan receive]:
github.com/alibaba/pouch/daemon/mgr.NewContainerMonitor.func1(0xc4202ce618)
        /tmp/pouchbuild/src/github.com/alibaba/pouch/daemon/mgr/container_monitor.go:62 +0x45
created by github.com/alibaba/pouch/daemon/mgr.NewContainerMonitor
        /tmp/pouchbuild/src/github.com/alibaba/pouch/daemon/mgr/container_monitor.go:60 +0x8d

goroutine 94 [chan receive]:
github.com/alibaba/pouch/daemon/mgr.(*ContainerManager).execProcessGC(0xc42037e090)
        /tmp/pouchbuild/src/github.com/alibaba/pouch/daemon/mgr/container.go:2177 +0x1a5
created by github.com/alibaba/pouch/daemon/mgr.NewContainerManager
        /tmp/pouchbuild/src/github.com/alibaba/pouch/daemon/mgr/container.go:179 +0x50b
```

goroutine stack 通常第一行包含着 Goroutine ID，接下来的几行是具体的调用栈信息。有了调用栈信息，我们就可以通过 __关键字匹配__ 的方式来检索是否存在泄漏的情况了。

在 Pouch 的集成测试里，Pouch Logs API 对包含 `(*Server).logsContainer` 的 goroutine stack 比较感兴趣。因此在测试跟随模式完毕后，会调用 `debug` 接口检查是否包含 `(*Server).logsContainer` 的调用栈。一旦发现包含便说明该 goroutine 还没有被回收，存在泄漏的风险。

总的来说，`debug` 接口的方式适用于 __集成测试__ ，因为测试用例和目标服务不在同一个进程里，需要 dump 目标进程的 goroutine stack 来获取泄漏信息。

### <a name="bsx8ai"></a>3.2 runtime.NumGoroutine

当测试用例和目标函数／服务在同一个进程里时，可以通过 goroutine 的数目变化来判断是否存在泄漏问题。

```go
func TestXXX(t *testing.T) {
    orgNum := runtime.NumGoroutine()
    defer func() {
        if got := runtime.NumGoroutine(); orgNum != got {
            t.Fatalf("xxx", orgNum, got)
        }
    }()

    ...
}
```


### <a name="d70iov"></a>3.3 github.com/google/gops

[gops](https://github.com/google/gops) 与包 `net/http/pprof` 相似，它是在你的进程内放入了一个 agent ，并提供命令行接口来查看进程运行的状态，其中 `gops stack ${PID}` 可以查看当前 goroutine stack 状态。

## <a name="3e7xce"></a>4. 小结

开发 HTTP Server 时，`net/http/pprof` 有助于我们分析代码情况。如果代码逻辑复杂、存在可能出现泄漏的情况时，不妨标记一些可能泄漏的函数，并将其作为测试中的一个环节，这样自动化 CI 就能在代码审阅前发现问题。

## <a name="h4f7xe"></a>5. 相关链接

* [Concurrency is not Parallelism](https://talks.golang.org/2012/waza.slide#1) 
* [Go Concurrency Patterns: Context](https://blog.golang.org/context) 
* [Profilling Go Programs](https://blog.golang.org/profiling-go-programs) 

