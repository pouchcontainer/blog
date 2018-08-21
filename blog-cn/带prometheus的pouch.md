# Prometheus加持的PouchContainer

`PouchContainer`通过`Prometheus`支持多种监测指标。我们现在已经有了基本的`golang`的运行环境和一些api延时指标。我们将在以下两个主要领域中增加更多的评测指标：
* 重要的`pouchd`评测指标
* 重要的api延时评测指标列表

## 如何增加新评测指标

我们倾向于在`PouchContainer`使用`prometheus`的[评测指标和标签命名约定](https://prometheus.io/docs/practices/naming/)。因此，当你要添加新的评测指标时，请遵循`prometheus`的评测指标和标签命名约定。

我们使用`promethus` 的[go-sdk](https://github.com/prometheus/client_golang) 去监测`pouchd`。它支持计数器、计量和概要等度量指标。请参考 [指标类型](https://prometheus.io/docs/concepts/metric_types/) 以获得更多信息。

## 如何使用

用户可通过`pouchd -l tcp://0.0.0.0:4243`指令启动`pouchd`来监听`0.0.0.0:4243`，然后发出`GET http://127.0.0.1:4243/metrics`请求以获取`prometheus`格式的评测指标的完整列表，具体如下：

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

这样，我们可以在`prometheus`中设置新目标以获得评价指标在每轮迭代中的取值。