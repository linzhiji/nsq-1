# 性能

## 分布式性能

主仓库包含一段代码（`bench/bench.py`），它能在 EC2 上自动完成分布式基准。

它引导 `N` 个节点，一些运行 `nsqd`，一些运行加载生成工具（`PUB` and
`SUB`），并分析它们的输出来提供聚合。		

## 初始化

下面的代码反应了默认参数6 `c3.2xlarge`，这个实例支持 1g 比特的连接。3 个节点运行 `nsqd`  实例，剩下的运行 `bench_reader` (`SUB`) and `bench_writer` (`PUB`) 实例，来生成依赖于基准模式的负载。

    $ ./bench/bench.py --access-key=... --secret-key=... --ssh-key-name=...
    [I 140917 10:58:10 bench:102] launching 6 instances
    [I 140917 10:58:12 bench:111] waiting for instances to launch...
    ...
    [I 140917 10:58:37 bench:130] (1) bootstrapping ec2-54-160-145-64.compute-1.amazonaws.com (i-0a018ce1)
    [I 140917 10:59:37 bench:130] (2) bootstrapping ec2-54-90-195-149.compute-1.amazonaws.com (i-0f018ce4)
    [I 140917 11:00:00 bench:130] (3) bootstrapping ec2-23-22-236-55.compute-1.amazonaws.com (i-0e018ce5)
    [I 140917 11:00:41 bench:130] (4) bootstrapping ec2-23-23-40-113.compute-1.amazonaws.com (i-0d018ce6)
    [I 140917 11:01:10 bench:130] (5) bootstrapping ec2-54-226-180-44.compute-1.amazonaws.com (i-0c018ce7)
    [I 140917 11:01:43 bench:130] (6) bootstrapping ec2-54-90-83-223.compute-1.amazonaws.com (i-10018cfb)

## 生产者吞吐量

这个基准仅反应了生产者吞吐量。消息体有 100 个字节，并且消息通过 3 个话题（topic）分布。

    $ ./bench/bench.py --access-key=... --secret-key=... --ssh-key-name=... --mode=pub --msg-size=100 run
    [I 140917 12:39:37 bench:140] launching nsqd on 3 host(s)
    [I 140917 12:39:41 bench:163] launching 9 producer(s) on 3 host(s)
    ...
    [I 140917 12:40:20 bench:248] [bench_writer] 10.002s - 197.463mb/s - 2070549.631ops/s - 4.830us/op

入口处 **`~2.07mm`** msgs/sec,使用了 **`197mb/s`** 的带宽。

## 生产和消费吞吐量

通过服务生产者和消费者，这个基准更加准确的反应了实际情况。这个消息也是 100 个字节，并且通过 3 个话题（topic）分布，每个都包含一个 通道（channel）（每个 通道（channel） 24 个客户端）。

    $ ./bench/bench.py --access-key=... --secret-key=... --ssh-key-name=... --msg-size=100 run
    [I 140917 12:41:11 bench:140] launching nsqd on 3 host(s)
    [I 140917 12:41:15 bench:163] launching 9 producer(s) on 3 host(s)
    [I 140917 12:41:22 bench:186] launching 9 consumer(s) on 3 host(s)
    ...
    [I 140917 12:41:55 bench:248] [bench_reader] 10.252s - 76.946mb/s - 806838.610ops/s - 12.706us/op
    [I 140917 12:41:55 bench:248] [bench_writer] 10.030s - 80.315mb/s - 842149.615ops/s - 11.910us/op

入口处的 **`~842k`**  **`~806k`** msgs/s, 合计消费带宽 **`156mb/s`**，我们已经尽力提升了 `nsqd` 节点的 CPU 处理能力。通过引入消费者，`nsqd` 需要维持每个 通道（channel），因此负载自然会高一点。


消费者的数量略微少于生产者，因为消费者发送2次命令（每个消息都要发送 `FIN` 命令）。

增加两个节点（一个是 `nsqd` 另一个是产生负载），达到了 **`1mm`** msgs/s：

    $ ./bench/bench.py --access-key=... --secret-key=... --ssh-key-name=... --msg-size=100 run
    [I 140917 13:38:28 bench:140] launching nsqd on 4 host(s)
    [I 140917 13:38:32 bench:163] launching 16 producer(s) on 4 host(s)
    [I 140917 13:38:43 bench:186] launching 16 consumer(s) on 4 host(s)
    ...
    [I 140917 13:39:12 bench:248] [bench_reader] 10.561s - 100.956mb/s - 1058624.012ops/s - 9.976us/op
    [I 140917 13:39:12 bench:248] [bench_writer] 10.023s - 105.898mb/s - 1110408.953ops/s - 9.026us/op

## 单个节点性能

声明：请牢记 **NSQ** 设计的初衷是分布式。单个节点的性能非常重要，但这并不是我们所追求的。

 * 2012 MacBook Air i7 2ghz
 * go1.2
 * NSQ v0.2.24
 * 200 byte messages

## GOMAXPROCS=1 (单个生产者，单个消费者)

{% highlight bash %}
$ ./bench.sh 
results...
PUB: 2014/01/12 22:09:08 duration: 2.311925588s - 82.500mb/s - 432539.873ops/s - 2.312us/op
SUB: 2014/01/12 22:09:19 duration: 6.009749983s - 31.738mb/s - 166396.273ops/s - 6.010us/op
{% endhighlight %}

## GOMAXPROCS=4 (4 个生产者, 4 个消费者)

{% highlight bash %}
$ ./bench.sh 
results...
PUB: 2014/01/13 16:58:05 duration: 1.411492441s - 135.130mb/s - 708469.965ops/s - 1.411us/op
SUB: 2014/01/13 16:58:16 duration: 5.251380583s - 36.321mb/s - 190426.114ops/s - 5.251us/op
{% endhighlight %}
