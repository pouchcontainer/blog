## <a name="pobkqe"></a>0. Introduction

[PouchContainer](https://github.com/alibaba/pouch) is an open-source project created by Alibaba Group. It provides strong isolation and high portability, which can help enterprises quickly containerize their stock business and improve the utilization of internal physical resources. 

PouchContainer is also a golang project in which goroutine has been widely used to implement management of various modules including containers, images, and logs. goroutine is an user-mode 'thread' supported by golang at the language level. This built-in feature can help developers build high-concurrency services at a rapid pace.

Though it's pretty handy to write concurrent programs with goroutine, it does bring a new problem often known as __goroutine leak__, which would occur when the receiver side of a channel was blocked for a long time but cannot wake up. goroutine leak is just as terrible as memory leak -- leaking goroutines constantly consume resources and slow down (or even break) the whole system. Hence developers should avoid goroutine leak to ensure the health of the system. This article will introduce the detection practice of goroutine leak used in PouchContainer project, by explaining the definition of goroutine leak and common analyzing tools used to detect the leak.

## <a name="toxmge"></a>1. Goroutine Leak

In the world of golang, there are many groundhogs at your disposal -- they can either handle a large amount of problems of the same type individually , or work together on one single task. And works can be done quickly as long as you give them the right order. As you may guess, these groundhogs are, as we often mentioned, goroutine. Each time you 'go', you will get a groundhog which can be ordered by you to execute some task.

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

In most cases, a groundhog will return to its cage after finishing the designated task, waiting for your next call. But chances are that this groundhog has not been returned for a very long time.

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

The above code will start an HTTP server which allows clients to execute shell commands remotely via HTTP requests. For instance, `curl "{ip}:8080/exec?cmd=ps&args=-ef"` can be used to check the processes running on the server. After the code has been executed, the groundhog will print logs to indicate the completion of execution.

However, in some cases, the request takes a really long time to process, while the requester doesn't have much patience to wait. Take this command as an example, `curl -m 3 "{ip}:8080/exec?cmd=dosomething"`, the requester will disconnect after 3 seconds regardless of the execution status. Since the above code didn't check the possible disconnection, if the requester choose to disconnect instead of waiting for the execution with patience, then this groundhog will not return its cage until the completion of execution. Then it comes the most horrible thing, think about this kind of command, `curl -m 1 "{ip}:8080/exec?cmd=sleep&args=10000"`, groundhogs won't be able to return their cage for a long time and will continuously consume system resources.

These wandering, uncontrolled groundhogs are, as we often mentioned, __goroutine leak__. There are many reasons for goroutine leak, such as a channel without sender. After running the following code, you will see that 2 goroutine will be shown constantly in runtime, one is 'main' function itself, the other is a groundhog waiting for data.

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

goroutine leak can happen in many different contexts. This following section will introduce how to detect goroutine leak and the corresponding solution to it, through the scenario of Pouch Logs API.

## <a name="eav8ss"></a>2. Pouch Logs API practice
### <a name="8rtcwv"></a>2.1 specific scenario

For better description of the problem, the code of Pouch Logs HTTP Handler has been simplified:

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

Logs API Handler will launch goroutine to read logs and send data to `writeLogStream` via channel, then `writeLogStream` will return data back to the caller. This Logs API provides a tracking functionality, which will continuously show new content of logs until the container stops. But note that the caller can terminate the request at any time, at its discretion. So how can we detect such remaining goroutine?

> Upon disconnection, the handler still tries to send data to client, which will cause an error of write: broken pipe, normally the goroutine will then exit. But if the handler is waiting data for a long time, it results in goroutine leak.

### <a name="6x6rvw"></a>2.2 How to detect goroutine leak？

For HTTP Server, we usually check the status of running processes by importing `net/http/pprof` package, in which there is one line used to check status of goroutine stack, `{ip}:{port}/debug/pprof/goroutine?debug=2`. Let's see what's the status of goroutine stack after the caller actively disconnect the server.

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

We can see that the `logsContainer` goroutine is still in the current running process. Since this container doesn't have any logs being output, this goroutine cannot exit via `write: broken pipe` error, instead it will consume system resources indefinitely. So how can we fix this problem?

### <a name="vhqywi"></a>2.3 How to fix it？

The `net/http` package in golang provides a feature to listen disconnection:

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

When the client disconnects before the request being processed, `CloseNotify()` will receive the corresponding message and cancel it via `context.Context`. In this way, we can fix the goroutine leak. In the world of golang, often times you will see goroutine for __read__ and __write__. This kind of function often has `context.Context` in their first argument, which can be used to reclaim goroutine via `WithTimeout` and `WithCancel`, avoiding any leaking.

> CloseNotify cannot be applied in the context of Hijack connection since after Hijack, everything related to the connection will be handled by the actual Handler. In other words, HTTP Server has relinquished the management of data.

So can this kind of detection be automated? The below section will demonstrate this with some common analyzing tools.

## <a name="shc8xg"></a>3. Common analyzing tools

### <a name="bshgpi"></a>3.1 net/http/pprof 

When developing an HTTP Server, we can turn on debug mode by importing `net/http/pprof` package, then access goroutine stack information via `/debug/pprof/goroutine`. Normally the goroutine stack is something like this:

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

The first line of goroutine stack usually contains Goroutine ID, and the next a few lines are detailed stacktrace. With stacktrace, we can search for potential leaking via __keyword matching__.

During the integration test of Pouch, Pouch Logs API pays additional attention to goroutine stack including `(*Server).logsContainer`. So as the tracking mode completes, it will invoke `debug` interface to check if there is any stacktrace containing `(*Server).logsContainer`. Once found such stacktrace, we know that some goroutines have not been reclaimed yet and may cause goroutine leak.

In conclusion, this way of using `debug` interface is more suitable for __integration test__, since the testing instance is not in the same process as the target service and need to dump the goroutine stack of target process to obtain leaking information.

### <a name="bsx8ai"></a>3.2 runtime.NumGoroutine

When the testing instance and target function/service are in the same process, number of goroutines can be used to detect leaking.

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

[gops](https://github.com/google/gops) is similar to `net/http/pprof` package. It puts an agent in your processes and provides command line interface for checking status of running processes. In particular, `gops stack ${PID}` can be used to check current state of goroutine stack.

## <a name="3e7xce"></a>4. Summary

When developing an HTTP Server, `net/http/pprof` is helpful for analyzing code. In case that your code has complicated logic or potential leaking exists, you can mark up some functions with potential leaks and take it as part of testing so that automated CI can detect problems before code review.

## <a name="h4f7xe"></a>5. Links

* [Original Doc](https://github.com/pouchcontainer/blog/blob/master/blog-cn/PouchContainer%20Goroutine%20Leak%20检测实践.md) 
* [Concurrency is not Parallelism](https://talks.golang.org/2012/waza.slide#1) 
* [Go Concurrency Patterns: Context](https://blog.golang.org/context) 
* [Profilling Go Programs](https://blog.golang.org/profiling-go-programs) 

