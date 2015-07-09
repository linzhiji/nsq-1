# 快速开始


下面的步骤将通过推送(publishing)、消费(cunsuming)和归档(archiving)消息到本地磁盘，在本地环境演示一个小型的 **NSQ** 集群

 1. 根据[文档安装][installing]安装 NSQ。

 2. 在另外一个shell中，运行 `nsqlookupd`:
        
        $ nsqlookupd

 3. 再开启一个shell，运行 `nsqd`:

        $ nsqd --lookupd-tcp-address=127.0.0.1:4160

 4. 再开启第三个shell，运行 `nsqadmin`:

        $ nsqadmin --lookupd-http-address=127.0.0.1:4161

 5. 开启第四个shell，推送一条初始化数据(并且在集群中创建一个topic):
 
        $ curl -d 'hello world 1' 'http://127.0.0.1:4151/put?topic=test'

 6. 最后，开启第五个shell， 运行 `nsq_to_file`:

        $ nsq_to_file --topic=test --output-dir=/tmp --lookupd-http-address=127.0.0.1:4161

 7. 推送更多地数据到 `nsqd`:

        $ curl -d 'hello world 2' 'http://127.0.0.1:4151/put?topic=test'
        $ curl -d 'hello world 3' 'http://127.0.0.1:4151/put?topic=test'

 8. 按照预先设想的，在浏览器中打开 `http://127.0.0.1:4171/` 就能查看 `nsqadmin` 的UI界面和队列统计数据。同时，还可以在 `/tmp` 目录下检查 (`test.*.log`) 文件.

这个教程中最重要的是：`nsq_to_file` (客户端)没有明确地指出 testtopic 被生产，它从 `nsqlookupd` 获取信息，即使在消息推送之后才开始连接 nsqd 消息也并没有消失。(这是里指保证每一条消息至少被一个消费者消费掉)。

[installing]: installing.md
