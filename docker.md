# Docker

这篇文章详细介绍了如何部署并在 [Docker](http://www.docker.com) 容器里运行 NSQ 二进制文件。

这有一个最小化的 NSQ 镜像文件，它包含了所有的 NSQ 二级制文件。每个二进制文件可以通过命令指定运行。基本格式如下：

    docker run nsqio/nsq /<command>

注意命令前的 `/`，例如:

    docker run nsqio/nsq /nsq_to_file

## 链接

* [docker](http://www.docker.com/)
* [`nsq` image](https://registry.hub.docker.com/u/nsqio/nsq/)

## 运行 nsqlookupd

    docker pull nsqio/nsq
    docker run --name lookupd -p 4160:4160 -p 4161:4161 nsqio/nsq /nsqlookupd

## 运行 nsqd

首先，得到 docker 主机 ip：

    ifconfig | grep addr

接着, 运行 `nsqd` 容器:

    docker pull nsqio/nsq
    docker run --name nsqd -p 4150:4150 -p 4151:4151 \
        nsqio/nsq /nsqd \
        --broadcast-address=<host> \
        --lookupd-tcp-address=<host>:<port>

设置 `--lookupd-tcp-address` 标志位到主机 IP，以及之前运行的 TCP 端口：


`nsqlookupd`, i.e. `dockerIP:4160`:

例如, 指定主机IP `172.17.42.1`:

    docker run --name nsqd -p 4150:4150 -p 4151:4151 \
        nsqio/nsq /nsqd \
        --broadcast-address=172.17.42.1 \
        --lookupd-tcp-address=172.17.42.1:4160

注意：这里使用端口 `4160`, 端口暴露了什么我们什么开始运行 `nsqlookupd` 容器 (它也是 `nsqlookupd` 的端口)。

如果你不想使用默认端口，改变 `-p` 参数:

    docker run --name nsqlookupd -p 5160:4160 -p 5161:4161 nsqio/nsq /nsqlookupd

它将会使得 nsqlookupd 在主机 IP 上的端口 5160 和 5161 可用。

## 使用 TLS

如果在 NSQ 容器上使用 TLS，你必须包含证书文件，私钥文件，和根 CA 文件。Docker 镜像里 `/etc/ssl/certs/` 包含这些内容。挂载一个主机文件夹包含这些文件，并在命令行里指定，比如：

    docker run -p 4150:4150 -p 4151:4151 -p 4152:4152 -v /home/docker/certs:/etc/ssl/certs \
        nsqio/nsq /nsqd \
        --tls-root-ca-file=/etc/ssl/certs/certs.crt \
        --tls-cert=/etc/ssl/certs/cert.pem \
        --tls-key=/etc/ssl/certs/key.pem \
        --tls-required=true \
        --tls-client-auth-policy=require-verify

上面的代码，运行的时候将会从 `/home/docker/certs` 里加载文件到 Docker 容器里。

## 持久化 NSQ 数据

使用 `/data` 目录来存储 `nsqd` 到主机磁盘上，它能让你加载到 [data-only Docker container](https://docs.docker.com/userguide/dockervolumes/#creating-and-mounting-a-data-volume-container) ，或者加载主机文件夹里:

    docker run nsqio/nsq /nsqd \
        --data-path=/data
