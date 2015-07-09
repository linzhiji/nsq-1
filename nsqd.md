# nsqd

`nsqd` 是一个守护进程，负责接收，排队，投递消息给客户端。

它可以独立运行，不过通常它是由 `nsqlookupd`  实例所在集群配置的（它在这能声明 topics 和 channels，以便大家能找到）。

它在 2 个 TCP 端口监听，一个给客户端，另一个是 HTTP API。同时，它也能在第三个端口监听 HTTPS。

### 命令行选项

    -auth-http-address=: <addr>:<port> 查询授权服务器 (可能会给多次)
    -broadcast-address="": 通过 lookupd  注册的地址（默认名是 OS）
    -config="": 配置文件路径
    -data-path="": 缓存消息的磁盘路径
    -deflate=true: 运行协商压缩特性（客户端压缩）
    -e2e-processing-latency-percentile=: 消息处理时间的百分比（通过逗号可以多次指定，默认为 none）
    -e2e-processing-latency-window-time=10m0s: 计算这段时间里，点对点时间延迟（例如，60s 仅计算过去 60 秒）
    -http-address="0.0.0.0:4151": 为 HTTP 客户端监听 <addr>:<port>
    -https-address="": 为 HTTPS 客户端 监听 <addr>:<port>
    -lookupd-tcp-address=: 解析 TCP 地址名字 (可能会给多次)
    -max-body-size=5123840: 单个命令体的最大尺寸
    -max-bytes-per-file=104857600: 每个磁盘队列文件的字节数
    -max-deflate-level=6: 最大的压缩比率等级（> values == > nsqd CPU usage)
    -max-heartbeat-interval=1m0s: 在客户端心跳间，最大的客户端配置时间间隔
    -max-message-size=1024768: (弃用 --max-msg-size) 单个消息体的最大字节数
    -max-msg-size=1024768: 单个消息体的最大字节数
    -max-msg-timeout=15m0s: 消息超时的最大时间间隔
    -max-output-buffer-size=65536: 最大客户端输出缓存可配置大小(字节）
    -max-output-buffer-timeout=1s: 在 flushing 到客户端前，最长的配置时间间隔。
    -max-rdy-count=2500: 客户端最大的 RDY 数量
    -max-req-timeout=1h0m0s: 消息从小排序的超时时间
    -mem-queue-size=10000: 内存里的消息数(per topic/channel)
    -msg-timeout="60s": 自动重新队列消息前需要等待的时间
    -snappy=true: 打开快速选项 (客户端压缩)
    -statsd-address="": 统计进程的 UDP <addr>:<port>
    -statsd-interval="60s": 从推送到统计的时间间隔
    -statsd-mem-stats=true: 切换发送内存和 GC 统计数据
    -statsd-prefix="nsq.%s": 发送给统计keys 的前缀(%s for host replacement)
    -sync-every=2500: 磁盘队列 fsync 的消息数
    -sync-timeout=2s: 每个磁盘队列 fsync 平均耗时
    -tcp-address="0.0.0.0:4150": TCP 客户端 监听的 <addr>:<port>
    -tls-cert="": 证书文件路径
    -tls-client-auth-policy="": 客户端证书授权策略 ('require' or 'require-verify')
    -tls-key="": 私钥路径文件
    -tls-required=false: 客户端连接需求 TLS
    -tls-root-ca-file="": 私钥证书授权 PEM 路径
    -verbose=false: 打开日志
    -version=false: 打印版本
    -worker-id=0: 进程的唯一码(默认是主机名的哈希值)

### HTTP API

 * [`/ping`](#ping) - 活跃度
 * [`/info`](#info) - 版本
 * [`/stats`](#stats) - 检查综合运行
 * [`/pub`](#pub) - 发布消息到话题（topic)
 * [`/mpub`](#mpub) - 发布多个消息到话题（topic)
 * [`/debug/pprof`](#debugpprof) - pprof 调试入口
 * [`/debug/pprof/profile`](#debugpprofprofile) - 生成 pprof CPU 配置文件
 * [`/debug/pprof/goroutine`](#debugpprofgoroutine) - 生成 pprof 计算配置文件
 * [`/debug/pprof/heap`](#debugpprofheap) - 生成 pprof 堆配置文件
 * [`/debug/pprof/block`](#debugpprofblock) - 生成 pprof 块配置文件
 * [`/debug/pprof/threadcreate`](#debugpprofthreadcreate) - 生成 pprof OS 线程配置文件

`v1` 命名空间 (as of `nsqd` `v0.2.29+`):

 * [`/topic/create`](#topiccreate) - 创建一个新的话题（topic)
 * [`/topic/delete`](#topicdelete) - 删除一个话题（topic)
 * [`/topic/empty`](#topicempty) - 清空话题（topic)
 * [`/topic/pause`](#topicpause) - 暂停话题（topic)的消息流
 * [`/topic/unpause`](#topicunpause) - 恢复话题（topic)的消息流
 * [`/channel/create`](#channelcreate) - 创建一个新的通道（channel)
 * [`/channel/delete`](#channeldelete) - 删除一个通道（channel)
 * [`/channel/empty`](#channelempty) - 清空一个通道（channel)
 * [`/channel/pause`](#channelpause) - 暂停通道（channel)的消息流
 * [`/channel/unpause`](#channelunpause) - 恢复通道（channel)的消息流

以抛弃的命名空间:

 * [`/create_topic`](#topiccreate) - 创建一个新的话题（topic)
 * [`/delete_topic`](#topicdelete) -  删除一个话题（topic)
 * [`/empty_topic`](#topicempty) - 清空话题（topic)
 * [`/pause_topic`](#topicpause) - 暂停话题（topic)的消息流
 * [`/unpause_topic`](#topicunpause) - 恢复话题（topic)的消息流
 * [`/create_channel`](#channelcreate) - 创建一个新的通道（channel)
 * [`/delete_channel`](#channeldelete) - 删除一个通道（channel)
 * [`/empty_channel`](#channelempty) - 清空一个通道（channel)
 * [`/pause_channel`](#channelpause) - 暂停通道（channel)的消息流
 * [`/unpause_channel`](#channelunpause) - 恢复通道（channel)的消息流

**NOTE**: 这些结束点返回 "wrapped" JSON:

    {"status_code":200, "status_text":"OK", "data":{...}}

发送 `Accept: application/vnd.nsq; version=1.0` 头将会协商使用未封装的 JSON 响应格式 (as of `nsqd` `v0.2.29+`).

#### /pub

发布一个消息

参数:

    topic - the topic to publish to
    
    POST body - the raw message bytes

{% highlight bash %}
$ curl -d "<message>" http://127.0.0.1:4151/pub?topic=message_topic`
{% endhighlight %}

#### /mpub

一个往返发布多个消息

参数:

    topic - 发布到的话题（topic)
    binary - bool ('true' or 'false') 允许二进制模式
    
    POST body - `\n` 分离原始消息字节

注意：默认的 `/mpub` 希望消息使用 `\n` 切割，使用 `?binary=true` 查询参数来允许二进制模式，希望发送的消息体能成为以下的格式（HTTP 'Content-Length' 头必须是将要发送的消息体的总大小）：

    [ 4-byte num messages ]
    [ 4-byte message #1 size ][ N-byte binary data ]
          ... (repeated <num_messages> times)

{% highlight bash %}
$ curl -d "<message>\n<message>\n<message>" http://127.0.0.1:4151/mpub?topic=message_topic`
{% endhighlight %}

#### /topic/create

已经抛弃的别名 `/create_topic`

创建一个话题（topic)

参数:

    话题（topic) - 将要创建的话题（topic)

#### /topic/delete

**已经抛弃的别名 :** `/delete_topic`

删除一个已经存在的话题（topic) (和所有的通道（channel))

参数:

    topic - 现有的话题（topic) to delete

#### /channel/create

**已抛弃的别名:** `/create_channel`

为现有的话题（topic)创建一个通道（channel)

参数:

    topic - 现有的话题（topic)
    channel - the channel to create

#### /channel/delete

**已抛弃的别名:** `/delete_channel`

删除现有的话题（topic)一个的通道（channel)

参数:

    topic - 现有的话题（topic)
    channel - 待删除的通道（channel)

#### /topic/empty

**已抛弃的别名:** `/empty_topic`

清空现有话题（topic)队列中所有的消息（内存和磁盘中）

参数:

    topic - 待清空的话题（topic)

#### /channel/empty

**已抛弃的别名:** `/empty_channel`

清空现有通道（channel)队列中所有的消息（内存和磁盘中）

参数:

    topic - 现有的话题（topic)
    channel - 待清空的通道（channel)

#### /topic/pause

**已抛弃的别名:** `/pause_topic`

暂停已有话题（topic)的所有通道（channel)的消息（消息将会在话题（topic)里排队）

参数:

    topic - 现有的话题（topic)

#### /topic/unpause

**已抛弃的别名:** `/unpause_topic`


为现有的话题（topic)的通道（channel)重启消息流

参数:

    topic - 现有的话题（topic)

#### /channel/pause

**已抛弃的别名:** `/channel_pause`

暂停发送已有的通道（channel)给消费者（消息将会队列）

参数:

    topic - 现有的话题（topic)
    channel - 已有的通道（channel)将会被暂停

#### /channel/unpause

**已抛弃的别名:** `/unpause_channel`

重新发送通道（channel)里的消息给消费者

参数:

    topic - 现有的话题（topic)
    channel - 将要暂停的通道（channel)

#### /stats

返回内部统计数据

参数

    format - (可选) `text` or `json` (默认 = `text`)

#### /ping

监控结束点，必须返回 `OK`。如果有问题返回 500。同时，如果写消息到磁盘失败将会返回错误状态。

#### /info

返回版本信息

#### /debug/pprof

可用的调试节点的页码

#### /debug/pprof/profile

开始 30秒的 `pprof` CPU 配置，并通过请求返回。

**注意，因为它在运行时的性能和时间，这个结束点并没在 `/debug/pprof` 页面列表中。**

#### /debug/pprof/goroutine

为所有运行的 goroutines 返回栈记录。

#### /debug/pprof/heap

返回堆和内存配置信息（前面的内容可作为 `pprof` 配置信息）

#### /debug/pprof/block

返回 goroutine 块配置信息

#### /debug/pprof/threadcreate

返回 goroutine 栈记录

### <a name="pprof">Debugging and Profiling</a>

`nsqd` 提供一套节点的配置信息， 直接通过 Go 的 [pprof][go_pprof] 工具。如果你有 go 工具套装，只要运行：

{% highlight bash %}
# memory profiling
$ go 工具 pprof http://localhost:4151/debug/pprof/heap

# cpu profiling
$ go 工具 pprof http://localhost:4151/debug/pprof/profile
{% endhighlight %}

### TLS

为了加强安全性，可以通过 `--tls-cert` 和 `--tls-key` 客户端配置 `nsqd`，升级他们的链接为 TLS。

另外，你可以要求客户端使用 `--tls-required` (`nsqd` `v0.2.28+`)协商 TLS。

你可以通过`--tls-client-auth-policy` (`require` 或 `require-verify`)配置一个 `nsqd` 客户端证书:

 * `require` - 客户端必须提供一个证书，否则将会被拒绝
 * `require-verify` - 客户端必须提供一个有效的证书，根据 `--tls-root-ca-file` 指定的链接或者默认的 CA，否则将会被拒绝。

可以当做客户端授权的表单（`nsqd` `v0.2.28+`）。

如果你想生成一个少密码，自签名证书，使用：

{% highlight bash %}
$ openssl req -x509 -newkey rsa:2048 -keyout key.pem -out cert.pem -days 365 -nodes
{% endhighlight %}

### <a name="auth">AUTH</a>

注意: 在 `nsqd` `v0.2.29+` 可用

通过使用一个遵从 Auth HTTP 协议的授权服务器，指定  `-auth-http-address=host:port` 标志，你可以配置 `nsqd`。

注意: 希望当仅有 `nsqd` TCP 协议暴露给外部客户端时使用授权，而不是 HTTP(S) 节点。参见底下说明： 


Auth  服务器必须接受 HTTP 请求：

    /auth?remote_ip=...&tls=...&auth_secret=...
    
返回结果格式如下：

    {
      "ttl": 3600,
      "identity": "username",
      "identity_url": "http://....",
      "authorizations": [
        {
          "permissions": [
            "subscribe",
            "publish"
          ],
          "topic": ".*",
          "channels": [
            ".*"
          ]
        }
      ]
    }

注意话题（topic)和通道（channel)字符串必须用 `nsqd` 的正则表达式来申请授权。`nsqd` 将会为 TTL 间隔，并会在这个间隔时间里重新请求授权。

通常情况，将会使用 TLS 来加强安全性。`nsqd` 和 授权服务器间通过信任的网络通信（并没被加密）。如果一个授权服务器通过远程 IP 信息来授权，客户端可以使用占位符（比如 `.`），作为 `AUTH` 命令（Auth 服务器忽略）。

授权服务器例子 [pynsqauthd](https://github.com/jehiah/nsqauth-contrib#pynsqauthd).

帮助服务器暴露 `nsqlookupd` 和 `nsqd` `/stats` 数据给客户端，从授权服务器通过权限过滤，在以下可以找到
[nsqauthfilter](https://github.com/jehiah/nsqauth-contrib#nsqauthfilter)

当使用命令行小工具，可以通过使用 `--reader-opt` 标志来授权。

{% highlight bash %}
$ nsq_tail ... -reader-opt="tls_v1,true" -reader-opt="auth_secret,$SECRET"
{% endhighlight %}

### <a name="e2e">点对点处理延迟</a>

你可以选择设置 `nsqd` 来收集和发射点对点信息处理延迟，通过 `--e2e-processing-latency-percentile` 标志位来配置百分比。

使用概率百分比技术（参见 *[Effective Computation of Biased Quantiles over Data Streams][bquant]*）来计算值。我们通过 [bmizerany][bmizerany]  来使用 [perks][perks] 包，它能实现这个算法。

我们内部维持2个通道（channel)，每个通道（channel)存储 `N/2` 分钟的延迟数据。每个 `N/2` 分钟我们重置了每个通道（channel)（并开始插入新的数据）。

因为我们仅在通道（channel)级别收集数据，对于话题（topic)我们聚合并合并所有的通道（channel)数量的 quantiles。如果数据在同一个 `nsqd` 实例上时，可以使用这个技术。然而当数据已经精确的通过 `nsqd` (通过 `nsqlookupd`），我们为每个 `nsqd` 取平均值。为了维持统计的精确性，除了平均值，我们也提供最大最小值。

注意: 如果没有消费者连接，不能更新值，尽管消息队列的点对点时间会缓慢增长。这是因为仅在  `nsqd`  收到从客户端发来  `FIN` 消息时才会重新计算。当消费者重新连接，这些值将会重新调整。


### <a name="statsd">Statsd / Graphite Integration</a>

当使用 `--statsd-address` 来为[statsd](https://github.com/etsy/statsd) （或类似 [statsdaemon](https://github.com/bitly/statsdaemon)）指定 UDP `<addr>:<port>` 时，`nsqd` 将会在 `--statsd-interval` 定期推送数据给 statsd（注意：这个间隔必须始终小于等于 graphite 的刷入间隔）。设置 `nsqadmin` 可以显示图标。

推荐以下配置（但是这些选择必须建立在你的可用资源和要求上）。同时，statsd 的刷入间隔必须小于或者等于 `storage-schemas.conf` 的最小值，并且 `nsqd` 必须通过 `--statsd-interval` 来确认刷入时间小于等于时间间隔。

{% highlight ini %}
# storage-schemas.conf
[nsq]
pattern = ^nsq\..*
retentions = 1m:1d,5m:30d,15m:1y

# storage-aggregation.conf
[default_nsq]
pattern = ^nsq\..*
xFilesFactor = 0.2 
aggregationMethod = average
{% endhighlight %}

`nsqd` 实例将会推送给以下 `statsd` 路径:

    nsq.<nsqd_host>_<nsqd_port>.topic.<topic_name>.backend_depth [gauge]
    nsq.<nsqd_host>_<nsqd_port>.topic.<topic_name>.depth [gauge]
    nsq.<nsqd_host>_<nsqd_port>.topic.<topic_name>.message_count
    nsq.<nsqd_host>_<nsqd_port>.topic.<topic_name>.channel.<channel_name>.backend_depth [gauge]
    nsq.<nsqd_host>_<nsqd_port>.topic.<topic_name>.channel.<channel_name>.clients [gauge]
    nsq.<nsqd_host>_<nsqd_port>.topic.<topic_name>.channel.<channel_name>.deferred_count [gauge]
    nsq.<nsqd_host>_<nsqd_port>.topic.<topic_name>.channel.<channel_name>.depth [gauge]
    nsq.<nsqd_host>_<nsqd_port>.topic.<topic_name>.channel.<channel_name>.in_flight_count [gauge]
    nsq.<nsqd_host>_<nsqd_port>.topic.<topic_name>.channel.<channel_name>.message_count
    nsq.<nsqd_host>_<nsqd_port>.topic.<topic_name>.channel.<channel_name>.requeue_count
    nsq.<nsqd_host>_<nsqd_port>.topic.<topic_name>.channel.<channel_name>.timeout_count
    
    # if --statsd-mem-stats is enabled
    nsq.<nsqd_host>_<nsqd_port>.mem.heap_objects [gauge]
    nsq.<nsqd_host>_<nsqd_port>.mem.heap_idle_bytes [gauge]
    nsq.<nsqd_host>_<nsqd_port>.mem.heap_in_use_bytes [gauge]
    nsq.<nsqd_host>_<nsqd_port>.mem.heap_released_bytes [gauge]
    nsq.<nsqd_host>_<nsqd_port>.mem.gc_pause_usec_100 [gauge]
    nsq.<nsqd_host>_<nsqd_port>.mem.gc_pause_usec_99 [gauge]
    nsq.<nsqd_host>_<nsqd_port>.mem.gc_pause_usec_95 [gauge]
    nsq.<nsqd_host>_<nsqd_port>.mem.mem.next_gc_bytes [gauge]
    nsq.<nsqd_host>_<nsqd_port>.mem.gc_runs

    # if --e2e-processing-latency-percentile is specified, for each percentile
    nsq.<nsqd_host>_<nsqd_port>.topic.<topic_name>.e2e_processing_latency_<percent> [gauge]
    nsq.<nsqd_host>_<nsqd_port>.topic.<topic_name>.channel.<channel_name>.e2e_processing_latency_<percent> [gauge]

[go_pprof]: http://golang.org/pkg/net/http/pprof/#pkg-overview
[bquant]: http://www.cs.rutgers.edu/~muthu/bquant.pdf
[bmizerany]: https://github.com/bmizerany
[perks]: https://github.com/bmizerany/perks
