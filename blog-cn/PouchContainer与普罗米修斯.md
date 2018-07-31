# PouchContainer与Prometheus

PouchContainer 通过[普罗米修斯](https://prometheus.io/)来支持各式各样的监视度量标准。 现在我们已经具备了基本的Go语言运行时和一些API延迟度量单位，我们计划未来在如下两个主要的方向增加新的机制:

* 重要的pouchd监视度量
* 在监视过程中的完整的API调用列表

## 如何添加新的衡量标准

在PouchContainer中我们倾向于使用普罗米修斯的 [度量和标签命名法则](https://prometheus.io/docs/practices/naming) 最佳实践. 所以当你需要添加新的度量时, 请务必遵循度量和标签命名法则。

我们使用普罗米修斯 [go-sdk](https://github.com/prometheus/client_golang) 来监视pouchd. 它支持计数器, 测量仪和总结性的测量类型. 如需获取更多信息, 请参阅 [度量类型](https://prometheus.io/docs/concepts/metric_types/).

## How to use

用户可以通过 `pouchd -l tcp://0.0.0.0:4243` 来在 `0.0.0.0:4243`启动pouchd监听, 然后发出`GET http://127.0.0.1:4243/metrics` 请求来获取普罗米修斯格式化的度量输出的完整列表

```
# HELP go_gc_duration_seconds A summary of the GC invocation durations.
# TYPE go_gc_duration_seconds summary
go_gc_duration_seconds{quantile="0"} 0.000111176
go_gc_duration_seconds{quantile="0.25"} 0.000198062
go_gc_duration_seconds{quantile="0.5"} 0.000269599
go_gc_duration_seconds{quantile="0.75"} 0.000474291
go_gc_duration_seconds{quantile="1"} 0.002013351
go_gc_duration_seconds_sum 0.021835193
go_gc_duration_seconds_count 52
# HELP go_goroutines Number of goroutines that currently exist.
# TYPE go_goroutines gauge
go_goroutines 22
# HELP go_info Information about the Go environment.
# TYPE go_info gauge
go_info{version="go1.9"} 1
...
# HELP http_request_size_bytes The HTTP request sizes in bytes.
# TYPE http_request_size_bytes summary
http_request_size_bytes{handler="prometheus",quantile="0.5"} NaN
http_request_size_bytes{handler="prometheus",quantile="0.9"} NaN
http_request_size_bytes{handler="prometheus",quantile="0.99"} NaN
http_request_size_bytes_sum{handler="prometheus"} 0
http_request_size_bytes_count{handler="prometheus"} 0
# HELP http_response_size_bytes The HTTP response sizes in bytes.
# TYPE http_response_size_bytes summary
http_response_size_bytes{handler="prometheus",quantile="0.5"} NaN
http_response_size_bytes{handler="prometheus",quantile="0.9"} NaN
http_response_size_bytes{handler="prometheus",quantile="0.99"} NaN
http_response_size_bytes_sum{handler="prometheus"} 0
http_response_size_bytes_count{handler="prometheus"} 0
# HELP pouch_image_pull_latency_microseconds Latency in microseconds to pull a image.
# TYPE pouch_image_pull_latency_microseconds summary
pouch_image_pull_latency_microseconds{image="docker.io/library/ubuntu:latest",quantile="0.5"} 3.7803132e+07
pouch_image_pull_latency_microseconds{image="docker.io/library/ubuntu:latest",quantile="0.9"} 3.7803132e+07
pouch_image_pull_latency_microseconds{image="docker.io/library/ubuntu:latest",quantile="0.99"} 3.7803132e+07
pouch_image_pull_latency_microseconds_sum{image="docker.io/library/ubuntu:latest"} 3.7803132e+07
pouch_image_pull_latency_microseconds_count{image="docker.io/library/ubuntu:latest"} 1
# HELP process_cpu_seconds_total Total user and system CPU time spent in seconds.
# TYPE process_cpu_seconds_total counter
process_cpu_seconds_total 4.78
# HELP process_max_fds Maximum number of open file descriptors.
# TYPE process_max_fds gauge
process_max_fds 1024
# HELP process_open_fds Number of open file descriptors.
# TYPE process_open_fds gauge
process_open_fds 9
# HELP process_resident_memory_bytes Resident memory size in bytes.
# TYPE process_resident_memory_bytes gauge
process_resident_memory_bytes 3.4521088e+07
# HELP process_start_time_seconds Start time of the process since unix epoch in seconds.
# TYPE process_start_time_seconds gauge
process_start_time_seconds 1.51064406778e+09
# HELP process_virtual_memory_bytes Virtual memory size in bytes.
# TYPE process_virtual_memory_bytes gauge
process_virtual_memory_bytes 4.91610112e+08
```

然后我们可以设置一个新的Target来在普罗米修斯中爬取这个度量点的数据即可完工。
