# 产品配置

虽然 `nsqd` 可以单独运行成节点，我们还是假设你希望使用分布式系统的优点。

以下是独立的二进制文件，需要安装并运行：

## nsqd

`nsqd` 是守护进程，接收，缓存，并投递消息给客户端

所有的配置都通过命令行参数来管理。我们希望默认配置能满足多数应用场景，有一部分参数值得注意：

`--mem-queue-size` 调整每个话题（topic)/通道（channnel）消息队列数。超过上限的消息，将会写到持平，通过 `--data-path` 定义。

同时, `nsqd` 将会需要通过 `nsqlookupd` 配置（参见以下详情），为每个实例指定参数。

拓扑结构，我们推荐运行 `nsqd` ，和生产消息服务共同写作。

`nsqd` 可以配置来推送数据到 [statsd][statsd]，通过指定 `--statsd-address`。在 `nsq.*` 命令空间里，`nsqd`发送统计数据，参见 [nsqd statsd][nsqd_statsd].

## nsqlookupd

`nsqlookupd` 是一个守护进程，为消费者提供运行时发现服务，来查找指定话题（topic）的生产者 `nsqd` 。

它维护非持久化状态，并且不需要和其他 `nsqlookupd` 实例来满足产线。

运行数据取决于你的冗余需求。使用很少的支援，并且可以和其他服务协同操作。我们推荐每个数据中心最少运行 3 个集群。

## nsqadmin

`nsqadmin` 是 web 服务，用来实时的管理你的 NSQ 集群。它通过和 `nsqlookupd` 实例交流，来确定生产者和 [graphite][graphite] 图表（要求打开 `nsqd` 端 `statsd`）。

我们仅需运行一个，并使它可以公开访问（安全）。

仅有一些 HTML 模板需要部署。默认 `nsqadmin`，位于  `/usr/local/share/nsqadmin/templates`，可以通过 `--template-dir` 重写。

要显示 `graphite` 图表，指定 `--graphite-url`。如果你已经使用 `statsd`，给所有的 keys 添加前缀，就需要指定 `--use-statsd-prefixes`。最后，如果 graphite 不能公开访问，通过指定`--proxy-graphite`， 你可以使用 `nsqadmin` 代理这些请求。

## Monitoring

每个守护进程，都拥有 `/ping` HTTP 端点，它可以用来创建监控检测。

即使实时调试，它也能运行的非常好：

    $ watch -n 0.5 "curl -s http://127.0.0.1:4151/stats"

一般通过 `nsqadmin` 进行调试，分析，管理。

[statsd]: https://github.com/bitly/statsdaemon
[graphite]: http://graphite.wikidot.com/
[nsqd_statsd]: ./nsqd.md#statsd
