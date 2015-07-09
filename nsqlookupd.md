# nsqlookupd

`nsqlookupd` 是守护进程负责管理拓扑信息。客户端通过查询  `nsqlookupd` 来发现指定话题（topic）的生产者，并且 `nsqd` 节点 广播话题（topic）和通道（channel）信息。

有两个接口：TCP 接口，`nsqd` 用它来广播。HTTP 接口，客户端用它来发现和管理。

### 命令行选项

    -http-address="0.0.0.0:4161": <addr>:<port> 监听 HTTP 客户端
    -inactive-producer-timeout=5m0s: 从上次 ping 之后，生产者驻留在活跃列表中的时长
    -tcp-address="0.0.0.0:4160": TCP 客户端监听的 <addr>:<port> 
    -broadcast-address: 这个 lookupd 节点的外部地址, (默认是 OS 主机名)
    -tombstone-lifetime=45s: 生产者保持 tombstoned  的时长
    -verbose=false: 允许输出日志
    -version=false: 打印版本信息

### HTTP 接口

#### /lookup

返回某个话题（topic）的生产者列表。

参数:

    topic - the 话题（topic） to list producers for

#### /topics

返回所有已知的话题（topic）

#### /channels

返回已知话题（topic）里的通道（channel）

参数:

    topic - the topic to list 通道（channel）s for

#### /nodes

返回所有已知的 `nsqd` 列表

#### /delete_topic

删除一个已存在的话题（topic）

参数:

    topic - 需要删除的话题（topic）

#### /delete_channel

删除一个已存在话题（topic）的通道（channel）

参数:

    topic - 已经存在的话题（topic）
    channel - 将要删除的通道（channel）

#### /tombstone_topic_producer

逻辑删除（Tombstones）某个话题（topic）的生产者。参见 [deletion and tombstones](#deletion_tombstones).

参数:

    topic - 已经存在的话题（topic）
    node - 将要逻辑删除（tombstones）的生产者(nsqd) (通过 <broadcast_address>:<http_port> 识别)

#### /ping

监控端点, 必须返回 `OK`

#### /info

返回版本信息

### <a name="deletion_tombstones">删除和逻辑删除（Tombstones）</a>

当一个话题（topic）不再全局生产，相对简单的操作是从集群里清理这个消息。假设所有的应用生产的消息下降，使用  `/delete_topic` 结束`nsqlookupd`  实例的，是必须要完成的操作。（内部来说，它将会识别 `nsqd` 生产者，并对这些节点执行合适的操作）。

全局来看，通道（channel）删除进程都很类似，不同点是你需用 `/delete_channel` 结束 `nsqlookupd` 实例，并且你必须保证所有的订阅了通道（channel）得消费者已经下降（downed）。

然而，当话题（topic）不再在节点的子集上生产的时候情况比较复杂。因为消费者查询 `nsqlookupd` 的方法并且连接到所有生产者，你加入的竞争环境尝试移除集群的信息，消费者发现这些节点并重新连接。（因此推送更新，话题（topic）仍然在节点上生产）。解决办法就是逻辑删除（tombstones）。逻辑删除（tombstones）在 `nsqlookupd` 上下文是特定的生产者和最后的配置 `--tombstone-lifetime` 时间。在这个窗口中，生产者不会在 `/lookup` 查询中列出，允许节点删除话题（topic），扩散这些信息到 `nsqlookupd`（接着逻辑删除（tombstoned）生产者），并阻止生产者重新发现这个节点。