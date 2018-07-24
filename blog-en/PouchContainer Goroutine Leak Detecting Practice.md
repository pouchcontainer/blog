## 0. Introduction

[PouchContainer](https://github.com/alibaba/pouch) is a container runtime product open sourced by Alibaba Group. With the features of strong isolation and portability, it can be used to help enterprises quickly containerize the inventory business and improve the utilization of internal physical resources.

PouchContainer is also a golang project, in which goroutine is heavily used to implement modules such as container management, mirror image management, and log management. Goroutine is a user-mode "thread" that golang supports at the language level, and this feature that natively support concurrency will help developers quickly build high concurrency services.

Although goroutine can perform concurrent or parallel operations easily, __goroutine leak__ will arise if there is a state in which the channel receiver is blocked for a long time but cannot be woken up. The goroutine leak is as terrible as a memory leak, and such a goroutine will continue to swallow resources, causing the system to run slower or even crash. In order for the system to function healthily, developers are required to ensure that goroutines do not leak. Next, this article will discuss the practice of goroutine leakage detection, from what is goroutine leak, how to detect it, and common profilers respectively.

## 1. Goroutine Leak

In golang, there are a lot of "marmots" in your control. They can handle a lot of identical issues simultaneously, and they can also collaborate on one thing. The problem can be quickly processed as long as you command properly. Yes, the marmot here is what we often say `goroutine`, you just have a `go`, you have a marmot, and it will perform the task you specified:

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

Normally, after a marmot completes the mission, it will return and wait for your next call. However, it is also possible that the marmot has not returned for a long time.

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
The above code will start the HTTP Server, which will allow the client to remotely execute shell commands via HTTP requests. For example, you can use `curl "{ip}:8080 / exec?cmd = ps&args = -ef"` to view the status of the server side process. After the execution, the marmot will print a log and indicate that the instruction has been executed.

But sometimes, the request requires the marmot to take a long time to process, and the requester has no patience to wait, such as `curl -m 3 "{ip}:8080/exec?cmd=dosomething"`, which says that the requester will disconnect the link in 3 seconds after executing a certain command. Since the above code does not detect the link disconnection, if the requester does not wait patiently for the command to complete but disconnects the link midway, the marmot will only be returned after the execution is completed. The scary thing is that when you encounter this `curl -m 1 "{ip}:8080/exec?cmd=sleep&args=10000"`, the marmots who can't get back in time will take up system resources.

These excavated, uncontrolled marmots are what we often say __goroutine leak__. There are many reasons for goroutine leak, channel has no sender, for example. After running the following code, you will find that the runtime will stably display a total of 2 goroutines, one of which is the `main` function itself, and the other is the marmot that has always been waiting for data.

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


There are many different scenarios for goroutine leaks. By describing the Pouch Logs API scenario, the following will introduce how to detect goroutine leaks and give solutions.

## 2. Pouch Logs API Practice

### 2.1 specific scenario

To better illustrate the problem, this article simplifies the code for the Pouch Logs HTTP Handler:

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

The Logs API Handler will start the goroutine to read the log and pass the data to `writeLogStream` using channel. `writeLogStream` will return the data to the caller. Logs API has the __following__ function, which will continuously display the new log content until the container stops. But for the caller who can terminate the request at any time, how to detect if there is any goroutine left?

> If the Handler still wants to send data to the client after the link is disconnected, there will be an error write: broken pipe, usually the goroutine will exit. But if the Handler still waits for data for a long time, it is a goroutine leak.

### 2.2 how to detect goroutine leak？


For HTTP Server, we usually check the status of the current process by importing the package `net/http/pprof`. One of the functions is to view the goroutine stack traces, `{ip}:{port}/debug/pprof/ Goroutine?debug=2`. Let's take a look at the goroutine stack information after the caller actively disconnected the link.

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

We will find that there is still a `logsContainer` goroutine in the current process. Because this container does not have any chance to output any logs, this goroutine can't exit with the error of `write: broken pipe`, it will always occupy system resources. So how can we solve this problem?

### 2.3 how to solve？

The package `net/http` provided by golang has the function to monitor link disconnection:

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

When the request has not been finished yet and the client actively quits, then `CloseNotify()` will receive the corresponding message and cancel it with `context.Context`, so we have a good handle on goroutine leak. In golang, you will often see __read__ and __write__ goroutine, and usually with `context.Context` in their first parameter, so you can control the recycling of goroutines by `WithTimeout` and `WithCancel`, and avoid leaks.

> CloseNotify is not applicable to Hijack linked scenarios, because after Hijack, all processing about the link is handed to the actual Handler, and HTTP Server has given up the management of the data.

So can such detecting be automated? The following will explain with common profilers.

## 3. Common profilers

### 3.1 net/http/pprof 

When developing HTTP Server, we can import the package `net/http/pprof` to toggle the debug mode on and access the goroutine stack information via `/debug/pprof/goroutine`. In general, the goroutine stack traces have the following style.

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

The goroutine stack usually contains the Goroutine ID in the first line, and the next few lines are the specific stackBackTrace, with which we can retrieve if there is a leak by __keyword matching__.

In the integration test, the Pouch Logs API is interested in the goroutine stack containing `(*Server).logsContainer`. So after the test-follow-mode is finished, the `debug` interface is called to check if the call stack for `(*Server).logsContainer` is included. Once the inclusion is found, it means that the goroutine has not been recycled and there is a risk of leakage.

In general, the `debug` interface is suitable for the __integration test__ because the test case and the target service are not in the same process, and the goroutine stack of the target process needs to be dumped to get the leak information.

### 3.2 runtime.NumGoroutine

When the test case and the target function/service are in the same process, the number of goroutines can be changed to determine if there is a leak.

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

### 3.3 github.com/google/gops

[gops](https://github.com/google/gops) is similar to the package `net/http/pprof`, which puts an agent inside your process and provides a command line interface to see which processes are running, where `gops stack ${PID}` can be used to view the current status of goroutine stack.

## 4. Summary

When developing the HTTP Server, `net/http/pprof` helps us analyze the code. If the code is logically complex or there is a potential for leaks, you might as well mark some functions which might leak and take them as part of the test so that the automated CI can spot problems before the code is reviewed.


## 5. Related Links

* [Concurrency is not Parallelism](https://talks.golang.org/2012/waza.slide#1) 
* [Go Concurrency Patterns: Context](https://blog.golang.org/context) 
* [Profilling Go Programs](https://blog.golang.org/profiling-go-programs) 

origin doc link: https://github.com/pouchcontainer/blog/blob/master/blog-cn/PouchContainer%20Goroutine%20Leak%20%E6%A3%80%E6%B5%8B%E5%AE%9E%E8%B7%B5.md
