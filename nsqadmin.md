# nsqadmin


`nsqadmin` 是一套 WEB UI，用来汇集集群的实时统计，并执行不同的管理任务。

## Command Line Options

    -graphite-url="": URL to graphite HTTP 地址
    -http-address="0.0.0.0:4171": <addr>:<port> HTTP clients 监听的地址和端口
    -lookupd-http-address=[]: lookupd HTTP 地址 (可能会提供多次)
    -notification-http-endpoint="": HTTP 端点 (完全限定) ，管理动作将会发送到
    -nsqd-http-address=[]: nsqd HTTP 地址 (可能会提供多次)
    -proxy-graphite=false: Proxy HTTP requests to graphite
    -template-dir="": 临时目录路径
    -use-statsd-prefixes=true: expect statsd prefixed keys in graphite (ie: 'stats_counts.')
    -version=false: 打印版本信息

## statsd / Graphite Integration


使用 `nsqd --statsd-address=...`的时候，你可以指定一个 `nsqadmin
--graphite-url=http://graphite.yourdomain.com` 允许 `nsqadmin` 上的graphite 图表. 如果使用一个统计克隆 (例如 [statsdaemon][statsdaemon])，它没有前缀的键值，也可以指定 `--use-statsd-prefix=false`.

## Admin 通知

如果设置了 `--notification-http-endpoint` 标志,每次 admin 动作执行的时候（例如暂停一个通道（channel））， `nsqadmin` 将会发送一个 POST 请求到指定(完全限定)端点。

请求的内容包含的动作信息，例如：

{% highlight json %}
{
  "action": "unpause_channel",
  "channel": "mouth",
  "topic": "beer",
  "timestamp": 1357683731,
  "user": "df",
  "user_agent": "Mozilla/5.0 (Macintosh; Iphone 8)"
  "remote_ip": "1.2.3.4:5678"
}
{% endhighlight %}

如果在请求时用户名可用，`user` 字段将会填充，如果之前使用 htpasswd 授权，或者[google-auth-proxy][gaproxy] 之后，否则卫空字符串。当不可用时 `channel` 字段也会为空。

提示: 通过设置 `--notification-http-endpoint` 为 `http://addr.of.nsqd/put?topic=admin_actions`，你可以创建一个 admin 的动作通知 NSQ 流，话题（topic）名为 `admin_actions`。

[gaproxy]: https://github.com/bitly/google_auth_proxy
[statsdaemon]: https://github.com/bitly/statsdaemon
