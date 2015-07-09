# 工具

These are utilities that facilitate common functionality and introspection into data streams.

## nsq_stat

为所有的话题（topic）和通道（channel）的生产者轮询 `/stats`，并显示统计数据：

    ---------------depth---------------+--------------metadata---------------
      total    mem    disk inflt   def |     req     t-o         msgs clients
      24660  24660       0     0    20 |  102688       0    132492418       1
      25001  25001       0     0    20 |  102688       0    132493086       1
      21132  21132       0     0    21 |  102688       0    132493729       1

## 命令行参数

    -channel="": NSQ 通道（channel）
    -lookupd-http-address=: lookupd HTTP 地址 (可能会给多次)
    -nsqd-http-address=: nsqd HTTP 地址 (可能会给多次)
    -status-every=2s: 轮询/打印输出见的时间间隔
    -topic="": NSQ 话题（topic）
    -version=false: 打印版本

## nsq_tail

消费指定的话题（topic）/通道（channel），并写到stdout (和 tail(1) 类似)

## 命令行参数

    -channel="": NSQ 通道（channel）
    -consumer-opt=: 传递给 nsq.Consumer (可能会给多次, http://godoc.org/github.com/bitly/go-nsq#Config)
    -lookupd-http-address=: lookupd HTTP 地址 (可能会给多次)
    -max-in-flight=200: 最大的消息数 to allow in flight
    -n=0: total messages to show (will wait if starved)
    -nsqd-tcp-address=: nsqd TCP 地址 (可能会给多次)
    -reader-opt=: (已经抛弃) 使用 --consumer-opt
    -topic="": NSQ 话题（topic）
    -version=false: 打印版本信息

## nsq\_to_file

消费指定的话题（topic）/通道（channel），并写道文件中，有选择的滚动和/或压缩文件。

## 命令行参数

    -channel="nsq_to_file": nsq 通道（channel）
    -consumer-opt=: 传递给 nsq.Consumer 的参数 (可能会给多次, http://godoc.org/github.com/bitly/go-nsq#Config)
    -datetime-format="%Y-%m-%d_%H": strftime，和 filename 里 <DATETIME> 格式兼容
    -filename-format="<TOPIC>.<HOST><GZIPREV>.<DATETIME>.log": output 文件名格式 (<TOPIC>, <HOST>, <DATETIME>, <GZIPREV> 重新生成. <GZIPREV> 是当已经存在的 gzip 文件的前缀)
    -gzip=false: gzip 输出文件
    -gzip-compression=3: (已经抛弃) 使用 --gzip-level, gzip 压缩级别(1 = 速度最佳, 2 = 最近压缩, 3 = 默认压缩)
    -gzip-level=6: gzip 压缩级别 (1-9, 1=BestSpeed, 9=BestCompression)
    -host-identifier="": 输出到 log 文件，提供主机名。 <SHORT_HOST> 和 <HOSTNAME> 是有效的替换者
    -lookupd-http-address=: lookupd HTTP 地址 (可能会给多次)
    -max-in-flight=200: 最大的消息数 to allow in flight
    -nsqd-tcp-address=: nsqd TCP 地址 (可能会给多次)
    -output-dir="/tmp": 输出文件所在的文件夹
    -reader-opt=: (已经抛弃) 使用 --consumer-opt
    -skip-empty-files=false: 忽略写空文件
    -topic=: nsq 话题（topic） (可能会给多次)
    -topic-refresh=1m0s: 话题（topic）列表刷新的频率是多少？
    -version=false: 打印版本信息

## nsq\_to_http

Consumes the specified 话题（topic）/通道（channel） and performs HTTP requests (GET/POST) to the specified
endpoints.

## 命令行擦拭你

    -channel="nsq_to_http": nsq 通道（channel）
    -consumer-opt=: 参数，通过 nsq.Consumer (可能会给多次, http://godoc.org/github.com/bitly/go-nsq#Config)
    -content-type="application/octet-stream": the Content-Type 使用d for POST requests
    -get=: HTTP 地址 to make a GET request to. '%s' will be printf replaced with data (可能会给多次)
    -http-timeout=20s: timeout for HTTP connect/read/write (each)
    -http-timeout-ms=20000: (已经抛弃) 使用 --http-timeout=X, timeout for HTTP connect/read/write (each)
    -lookupd-http-address=: lookupd HTTP 地址 (可能会给多次)
    -max-backoff-duration=2m0s: (已经抛弃) 使用 --consumer-opt=max_backoff_duration,X
    -max-in-flight=200: 最大的消息数 to allow in flight
    -mode="round-robin": the upstream request mode options: multicast, round-robin, hostpool
    -n=100: number of concurrent publishers
    -nsqd-tcp-address=: nsqd TCP 地址 (可能会给多次)
    -post=: HTTP 地址 to make a POST request to.  data will be in the body (可能会给多次)
    -reader-opt=: (已经抛弃) 使用 --consumer-opt
    -round-robin=false: (已经抛弃) 使用 --mode=round-robin, enable round robin mode
    -sample=1: % of messages to publish (float b/w 0 -> 1)
    -status-every=250: the # of requests between logging status (per handler), 0 disables
    -throttle-fraction=1: (已经抛弃) 使用 --sample=X, publish only a fraction of messages
    -topic="": nsq 话题（topic）
    -version=false: 打印版本信息

## nsq\_to_nsq

Consumes the specified 话题（topic）/channel and re-publishes the messages to destination `nsqd` via TCP.

## 命令行参数

    -channel="nsq_to_nsq": nsq 通道（channel）
    -consumer-opt=: 参数，通过 nsq.Consumer (可能会给多次, see http://godoc.org/github.com/bitly/go-nsq#Config)
    -destination-nsqd-tcp-address=: destination nsqd TCP 地址 (可能会给多次)
    -destination-topic="": destination nsq 话题（topic）
    -lookupd-http-address=: lookupd HTTP 地址 (可能会给多次)
    -max-backoff-duration=2m0s: (已经抛弃) 使用 --consumer-opt=max_backoff_duration,X
    -max-in-flight=200: 允许 flight 最大的消息数
    -mode="round-robin": 上行请求的参数: round-robin (默认), hostpool
    -nsqd-tcp-address=: nsqd TCP 地址 (可能会给多次)
    -producer-opt=: 传递到 nsq.Producer (可能会给多次, 参见 http://godoc.org/github.com/bitly/go-nsq#Config)
    -reader-opt=: (已经抛弃) 使用 --consumer-opt
    -require-json-field="": JSON 消息: 仅传递消息，包含这个参数
    -require-json-value="": JSON 消息: 仅传递消息要求参数有这个值 
    -status-every=250: # 请求日志的状态(每个目的地), 0 不可用
    -topic="": nsq 话题（topic）
    -version=false: 打印版本信息
    -whitelist-json-field=: JSON 消息: 传递这个字段 (可能会给多次)

## to_nsq

采用 stdin 流，并分解到新行（默认），通过 TCP 重新发布到目的地 `nsqd`。

## 命令行参数

    -delimiter="\n": 分割字符串(默认'\n')
    -nsqd-tcp-address=: 目的地 nsqd TCP 地址 (可能会给多次)
    -producer-opt=: 参数，通过 nsq.Producer (可能会给多次, http://godoc.org/github.com/bitly/go-nsq#Config)
    -topic="": 发布到的 NSQ 话题（topic）
