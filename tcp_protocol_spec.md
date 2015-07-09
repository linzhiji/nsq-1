# TCP 协议规范

NSQ 协议足够简单，用任何语言编译客户端都很容易。我们提供官方的 Go 和 Python 客户端库。

`nsqd` 进程通过监听配置的 TCP 端口来接受客户端连接。

连接后，客户端必须发送一个 4 字节的"魔术"标识码，表示通讯协议的版本。

 * `V2` (4个字节的 ASCII `[space][space][V][2]`)
   消费用到的推送流协议（和发布用到的请求/响应协议）

认证后，客户端可以发送 `IDENTIFY` 命令来停供常用的元数据（比如，更多的描述标识码）和协商特性。为了消费消息，客户端必须`SUB` 到一个通道（channel)。

订阅的时候，客户端的 `RDY` 状态卫 0。意味着没有消息会被发送到客户端。当客户端已经准备好接受消息时，需要把 `RDY` 设置为 #。比如设置为 100，不需要任何附加命令，将会有 100 条消息推送到客户端（每次服务端都会相应的减少 `RDY` 的值）。

V2 版本的协议让客户端拥有心跳功能。每隔 30 秒（默认设置），`nsqd` 将会发送一个 `_heartbeat_` 响应，并期待返回。如果客户端空闲，发送 `NOP`命令。如果 2 个 `_heartbeat_` 响应没有被应答， `nsqd` 将会超时，并且强制关闭客户端连接。 `IDENTIFY`  命令可以用来改变/禁用这个行为。

## <a name="注意s">节点（注意s）</a>

 * 除非 stated  ，所有的传输的二级制大小/整数都是网络字节顺序。(列如. *big* endian)

 * 有效的*话题（topic)*和*通道（channel)*名必须是字符`[.a-zA-Z0-9_-]` 和数字 `1 < length <= 64` (在 `nsqd` `0.2.28` 版本前最长 `32` 位)

## 命令

## <a name="identify">IDENTIFY</a>

更新服务器上的客户端元数据和协商功能。

    IDENTIFY\n
    [ 4-byte size in bytes ][ N-byte JSON data ]

注意: 这个命令包含 **JSON** 的相关内容，包括：

 * **`short_id`** (`nsqd` `v0.2.28+` 版本之后已经抛弃，使用 **`client_id`** 替换)这个标示符是描述的简易格式（比如，主机名）

 * **`long_id`** (`v0.2.28+` 版之后已经抛弃，使用**`hostname` 替换)这个标示符是描述的长格式。(比如. 主机名全名)

 * **`client_id`** 这个标示符用来消除客户端的歧义 (比如. 一些指定给消费者)

 * **`hostname`** 部署了客户端的主机名

 * **`feature_negotiation`** (`nsqd` `v0.2.19+`) bool， 用来标示客户端支持的协商特性。如果服务器接受，将会以 JSON 的形式发送支持的特性和元数据。

 * **`heartbeat_interval`** (`nsqd` `v0.2.19+`) 心跳的毫秒数.

     有效范围: `1000 <= heartbeat_interval <= configured_max` (`-1` 禁用心跳)

     `--max-heartbeat-interval` (nsqd 标志位) 控制最大值

     默认值 `--client-timeout / 2`

 * **`output_buffer_size`** (`nsqd` `v0.2.21+`) 当 nsqd 写到这个客户端时将会用到的缓存的大小（字节数）。

     有效范围: `64 <= output_buffer_size <= configured_max` (`-1` 禁用输出缓存)

     `--max-output-buffer-size` (nsqd 标志位) 控制最大值

     默认值 `16kb`

 * **`output_buffer_timeout`** (`nsqd` `v0.2.21+`) the timeout after which any data that nsqd has
                                 buffered will be flushed to this client.

     有效范围: `1ms <= output_buffer_timeout <= configured_max` (`-1` 禁用 timeouts)

     `--max-output-buffer-timeout` (nsqd 标志位) 控制最大值

     默认值 `250ms`

     **警告**: 使用极小值 `output_buffer_timeout` (`< 25ms`) 配置客户端，将会显著提高 `nsqd` CPU 的使用率（通常客户端连接时 `> 50` ）

     这依赖于 Go 的 timers 的实现，它通过 Go 的优先队列运行时间维护。细节参见 [pull request #236][pull_req_236] 里的 [commit message][043b79ac]。

 * **`tls_v1`** (`nsqd` `v0.2.22+`) 允许 TLS for this connection.

     `--tls-cert` and `--tls-key` (nsqd 标志位s) 允许 TLS 并配置服务器证书

     如果服务器支持 TLS，将会回复 `"tls_v1": true`。

     客户端读取 `IDENTIFY` 响应后，必须立即开始 TLS 握手。

     完成 TLS 握手后服务器将会响应 `OK`.

 * **`snappy`** (`nsqd` `v0.2.23+`) 允许 snappy 压缩这次连接

    `--snappy` (nsqd 标志位) 允许服务端支持

	 客户端不允许同时 `snappy` 和 `deflate`。

 * **`deflate`** (`nsqd` `v0.2.23+`) 允许 deflate 压缩这次连接

    `--deflate` (nsqd 标志位) 允许服务端支持

    客户端不允许同时 `snappy` 和 `deflate`。

 * **`deflate_level`** (`nsqd` `v0.2.23+`) 配置 deflate 压缩这次连接的级别

    `--max-deflate-level` (nsqd 标志位) 配置允许的最大值

    有效范围: `1 <= deflate_level <= configured_max`

    值越高压缩率越好，但是 CPU 负载也高。

 * **`sample_rate`** (`nsqd` `v0.2.25+`) 投递此次连接的消息接收率。

    有效范围: `0 <= sample_rate <= 99` (`0` 禁用)

    默认值 `0`

 * **`user_agent`** (`nsqd` `v0.2.25+`) 这个客户端的代理字符串

    默认值: `<client_library_name>/<version>`

 * **`msg_timeout`** (`nsqd` `v0.2.28+`) 配置服务端发送消息给客户端的超时时间

成功后响应：

    OK

注意: 如果客户端发送了 `feature_negotiation` (并且服务端支持)，响应体将会是 JSON 。

错误后的响应内容:

    E_INVALID
    E_BAD_BODY

## SUB

订阅话题（topic)/通道（channel)

    SUB <topic_name> <channel_name>\n

    <topic_name> - 字符串 (建议包含 #ephemeral 后缀)
    <channel_name> - 字符串 (建议包含 #ephemeral 后缀)

成功后响应:

    OK

错误后响应:

    E_INVALID
    E_BAD_TOPIC
    E_BAD_CHANNEL

## PUB

发布一个消息到 **话题（topic)**:

    PUB <topic_name>\n
    [ 4-byte size in bytes ][ N-byte binary data ]

    <topic_name> - 字符串 (建议 having #ephemeral suffix)

成功后响应:

    OK

错误后响应:

    E_INVALID
    E_BAD_TOPIC
    E_BAD_MESSAGE
    E_PUB_FAILED

## MPUB

发布多个消息到 **话题（topic)** (自动):

注意: `nsqd` `v0.2.16+` 有效

    MPUB <topic_name>\n
    [ 4-byte body size ]
    [ 4-byte num messages ]
    [ 4-byte message #1 size ][ N-byte binary data ]
          ... (repeated <num_messages> times)

    <topic_name> - 字符串 (建议 having #ephemeral suffix)

成功后响应:

    OK

错误后响应:

    E_INVALID
    E_BAD_TOPIC
    E_BAD_BODY
    E_BAD_MESSAGE
    E_MPUB_FAILED

## RDY

更新 `RDY` 状态 (表示你已经准备好接收`N` 消息)

注意: `nsqd` `v0.2.20+` 使用 `--max-rdy-count` 表示这个值

    RDY <count>\n

    <count> - a string representation of integer N where 0 < N <= configured_max

注意: 这个没有成功后响应

错误后响应:

    E_INVALID

## FIN

完成一个消息 (表示成功处理)

    FIN <message_id>\n

    <message_id> - message id as 16-byte hex string

注意: 这里没有成功后响应

错误后响应:

    E_INVALID
    E_FIN_FAILED

## REQ

重新将消息队列（表示处理失败）

这个消息放在队尾，表示已经发布过，但是因为很多实现细节问题，不要严格信赖这个，将来会改进。

简单来说，消息在传播途中，并且超时就表示 `REQ`。

    REQ <message_id> <timeout>\n

    <message_id> - message id as 16-byte hex string
    <timeout> - a string representation of integer N where N <= configured max timeout
        0 is a special case that will not defer re-queueing

注意: 这里没有成功后响应

错误后响应:

    E_INVALID
    E_REQ_FAILED

## TOUCH

重置传播途中的消息超时时间

注意: 在 `nsqd` `v0.2.17+` 可用

    TOUCH <message_id>\n

    <message_id> - the hex id of the message

注意: 这里没有成功后响应

错误后响应:

    E_INVALID
    E_TOUCH_FAILED

## CLS

清除连接（不再发送消息）

    CLS\n

成功后响应s:

    CLOSE_WAIT

错误后响应:

    E_INVALID

## NOP

No-op

    NOP\n

注意: 这里没有response

## AUTH

注意: 在 `nsqd` `v0.2.29+` 可用

如果 `IDENTIFY` 响应中有 `auth_required=true`，客户端必须在 `SUB`, `PUB` 或 `MPUB` 命令前前发送 `AUTH` 。否则，客户端不需要认证。

当 `nsqd` 接收到 `AUTH` 命令，它通过执行 HTTP 配置 `--auth-http-address` ，这个请求包括以下查询参数：连接的远程地址，TLS 状态，支持的认证密码。更多细节参见：[AUTH][nsqd_auth]

    AUTH\n
    [ 4-byte size in bytes ][ N-byte Auth Secret ]

成功后响应:

JSON 包含授权给客户端的身份，可选的 URL，和授权过的权限列表。

    {"identity":"...", "identity_url":"...", "permission_count":1}

错误后响应:

    E_AUTH_FAILED - An error occurred contacting an auth server
    E_UNAUTHORIZED - No permissions found


## 数据格式

数据异步传输给客户端，并且支持各种回复体，比如

    [x][x][x][x][x][x][x][x][x][x][x][x]...
    |  (int32) ||  (int32) || (binary)
    |  4-byte  ||  4-byte  || N-byte
    ------------------------------------...
        size     frame type     data

客户端必须是以下类型之一：

    FrameTypeResponse int32 = 0
    FrameTypeError    int32 = 1
    FrameTypeMessage  int32 = 2

以及消息格式:

    [x][x][x][x][x][x][x][x][x][x][x][x][x][x][x][x][x][x][x][x][x][x][x][x][x][x][x][x][x][x]...
    |       (int64)        ||    ||      (hex string encoded in ASCII)           || (binary)
    |       8-byte         ||    ||                 16-byte                      || N-byte
    ------------------------------------------------------------------------------------------...
      nanosecond timestamp    ^^                   message ID                       message body
                           (uint16)
                            2-byte
                           attempts

[pull_req_236]: https://github.com/bitly/nsq/pull/236
[043b79ac]: https://github.com/mreiferson/nsq/commit/043b79acda5fe57056b3cc21b2ef536d5615a2c2]
[nsqd_auth]: {{ site.baseurl }}/components/nsqd.html
