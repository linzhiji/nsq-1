# 安装

## <a name="binary">Binary Releases</a>

为 linux 和 darwin 预编译二进制文件 (`nsqd`, `nsqlookupd`, `nsqadmin`,以及所有的例子应用)，可用来下载。

## 当前稳定 Release 版本: **`v0.3.5`**

 * [nsq-0.3.5.darwin-amd64.go1.4.2.tar.gz][0.3.5_darwin_go142]
 * [nsq-0.3.5.linux-amd64.go1.4.2.tar.gz][0.3.5_linux_go142]

## 老的稳定 Release  版本

 * [nsq-0.3.2.darwin-amd64.go1.4.1.tar.gz][0.3.2_darwin_go141]
 * [nsq-0.3.2.linux-amd64.go1.4.1.tar.gz][0.3.2_linux_go141]
 * [nsq-0.3.1.darwin-amd64.go1.4.1.tar.gz][0.3.1_darwin_go141]
 * [nsq-0.3.1.linux-amd64.go1.4.1.tar.gz][0.3.1_linux_go141]
 * [nsq-0.3.0.darwin-amd64.go1.3.3.tar.gz][0.3.0_darwin_go133]
 * [nsq-0.3.0.linux-amd64.go1.3.3.tar.gz][0.3.0_linux_go133]
 * [nsq-0.2.31.darwin-amd64.go1.3.1.tar.gz][0.2.31_darwin_go131]
 * [nsq-0.2.31.linux-amd64.go1.3.1.tar.gz][0.2.31_linux_go131]

## Docker

参见 [the docs][docker_docs] 使用[Docker][docker]部署 NSQ。

## OSX

     $ brew install nsq

## 从源文件编译

## Pre-requisites

 * **[golang](http://golang.org/doc/install)** (version **`1.2+`** is required)
 * **[gpm](https://github.com/pote/gpm)** (dependency manager)

## 编译

NSQ 使用 **[gpm](https://github.com/pote/gpm)** 来管理依赖文件。 **编译源文件，`gpm` 是首选方案。**

{% highlight bash %}
$ gpm install
$ go get github.com/bitly/nsq/...
{% endhighlight %}

NSQ 保持了 `go get` 兼容，但是不推荐，因为之后不能保证仍然能稳定编译。

## Testing

{% highlight bash %}
$ ./test.sh
{% endhighlight %}

[0.3.5_darwin_go142]: https://s3.amazonaws.com/bitly-downloads/nsq/nsq-0.3.5.darwin-amd64.go1.4.2.tar.gz
[0.3.5_linux_go142]: https://s3.amazonaws.com/bitly-downloads/nsq/nsq-0.3.5.linux-amd64.go1.4.2.tar.gz

[0.3.2_darwin_go141]: https://s3.amazonaws.com/bitly-downloads/nsq/nsq-0.3.2.darwin-amd64.go1.4.1.tar.gz
[0.3.2_linux_go141]: https://s3.amazonaws.com/bitly-downloads/nsq/nsq-0.3.2.linux-amd64.go1.4.1.tar.gz

[0.3.1_darwin_go141]: https://s3.amazonaws.com/bitly-downloads/nsq/nsq-0.3.1.darwin-amd64.go1.4.1.tar.gz
[0.3.1_linux_go141]: https://s3.amazonaws.com/bitly-downloads/nsq/nsq-0.3.1.linux-amd64.go1.4.1.tar.gz

[0.3.0_darwin_go133]: https://s3.amazonaws.com/bitly-downloads/nsq/nsq-0.3.0.darwin-amd64.go1.3.3.tar.gz
[0.3.0_linux_go133]: https://s3.amazonaws.com/bitly-downloads/nsq/nsq-0.3.0.linux-amd64.go1.3.3.tar.gz

[0.2.31_darwin_go131]: https://s3.amazonaws.com/bitly-downloads/nsq/nsq-0.2.31.darwin-amd64.go1.3.1.tar.gz
[0.2.31_linux_go131]: https://s3.amazonaws.com/bitly-downloads/nsq/nsq-0.2.31.linux-amd64.go1.3.1.tar.gz

[docker]: https://docker.io/
[docker_docs]: ./docker.md
