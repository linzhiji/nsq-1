### 介绍

**NSQ** 是分布式实时的系统，其设计的目的是用来放大，处理每天数以十亿计级别的消息。

NSQ 具有*分布式*和*去中心化*拓扑结构，该结构具有无单点故障、故障容错、高可用性以及能够保证消息的可靠传递的特征。 参见 [features & guarantees][features_guarantees].

NSQ 非常容易配置和部署，且具有最大的灵活性，支持众多消息协议。另外，官方还提供了拆箱即用 Go 和 Python 库。如果读者兴趣构建自己的客户端的话，还可以参考官方提供的[协议规范][protocol]。

### 产品

<center><table class="production"><tr>
<td align="center"><a href="http://bitly.com"><img src="images//bitly_logo.png" width="84"/></a></td>
<td align="center"><a href="http://life360.com"><img src="images//life360_logo.png" width="100"/></a></td>
<td align="center"><a href="http://hailocab.com"><img src="images//hailo_logo.png" width="62"/></a></td>
<td align="center"><a href="http://simplereach.com"><img src="images//simplereach_logo.png" width="136"/></a></td>
<td align="center"><a href="http://moz.com"><img src="images//moz_logo.png" width="108"/></a></td>
<td align="center"><a href="http://path.com"><img src="images//path_logo.png" width="84"/></a></td>
</tr><tr>
<td align="center"><a href="http://segment.io"><img src="images//segmentio_logo.png" width="70"/></a></td>
<td align="center"><a href="http://eventful.com"><img src="images//eventful_logo.png" width="95"/></a></td>
<td align="center"><a href="http://energyhub.com"><img src="images//energyhub_logo.png" width="99"/></a></td>
<td align="center"><a href="https://project-fifo.net"><img src="images//project_fifo.png" width="97"/></a></td>
<td align="center"><a href="http://trendrr.com"><img src="images//trendrr_logo.png" width="97"/></a></td>
<td align="center"><a href="http://reonomy.com"><img src="images//reonomy_logo.png" width="100"/></a></td>
</tr><tr>
<td align="center"><a href="http://hw-ops.com"><img src="images//heavy_water.png" width="50"/></a></td>
<td align="center"><a href="http://lytics.io"><img src="images//lytics.png" width="100"/></a></td>
<td align="center"><a href="http://mediaforge.com"><img src="images//rakuten.png" width="100"/></a></td>
<td align="center"><a href="http://socialradar.com"><img src="images//socialradar_logo.png" width="100"/></a></td>
<td align="center"><a href="http://wistia.com"><img src="images//wistia_logo.png" width="140"/></a></td>
<td align="center"><a href="http://stripe.com"><img src="images//stripe_logo.png" width="140"/></a></td>
</tr><tr>
<td align="center"><a href="http://soundest.com"><img src="images//soundest_logo.png" width="140"/></a></td>
<td align="center"><a href="http://docker.com"><img src="images//docker_logo.png" width="145"/></a></td>
<td align="center"><a href="http://getweave.com"><img src="images//weave_logo.png" width="110"/></a></td>
<td align="center"><a href="http://shipwire.com"><img src="images//shipwire_logo.png" width="140"/></a></td>
<td align="center"><a href="http://digg.com"><img src="images//digg_logo.png" width="140"/></a></td>
<td align="center"><a href="http://scalabull.com"><img src="images//scalabull_logo.png" width="110"/></a></td>
</tr><tr>
<td align="center"><a href="http://augury.com"><img src="images//augury_logo.png" width="110"/></a></td>
<td align="center:"><a href="http://buzzfeed.com"><img src="images//buzzfeed_logo.png" width="97"/></a></td>
<td align="center"><a href="http://eztable.com"><img src="images//eztable_logo.png" width="105"/></a></td>
<td align="center"><a href="http://www.dotabuff.com"><img src="images//dotabuff_logo.png" width="105"/></a></td>
</tr></table></center>

### Twitter

<center>
<a class="twitter-timeline" width="520" height="600" data-dnt="true" data-chrome="noborders noheader" href="https://twitter.com/nsqio/favorites" data-widget-id="535600226652782593">Favorite Tweets by @nsqio</a>
<script>!function(d,s,id){var js,fjs=d.getElementsByTagName(s)[0],p=/^http:/.test(d.location)?'http':'https';if(!d.getElementById(id)){js=d.createElement(s);js.id=id;js.src=p+"://platform.twitter.com/widgets.js";fjs.parentNode.insertBefore(js,fjs);}}(document,"script","twitter-wjs");</script>
</center>

### 帮助

* **源码**: [https://github.com/bitly/nsq][github_nsq]
* **问题**: [https://github.com/bitly/nsq/issues][github_issues]
* **邮件组**: [nsq-users@googlegroups.com][google_group]
* **IRC**: #nsq on freenode
* **Twitter**: [@nsqio][nsqio_twitter]

### 问题

所有的问题必须通过 [github issues][github_issues] 提交。提交前搜索一下之前的问题，避免重复。 

### 贡献

NSQ 拥有一个增长的社区，欢迎贡献者（尤其是文档方面）。大家可以通过 fork 工程 [github][github_nsq] 并提交 pull 请求。

### 致谢

NSQ 是由 Matt Reiferson ([@imsnakes][snakes_twitter]) 和 Jehiah Czebotar
([@jehiah][jehiah_twitter]) 设计开发的，同时也离不开 [bitly][bitly]
和所有 [contributors][contributors] 贡献者们.

[features_guarantees]: features_and_guarantees.md
[protocol]: tcp_protocol_spec.md
[client_libraries]: client_libraries.md
[github_issues]: https://github.com/bitly/nsq/issues
[github_nsq]: http://github.com/bitly/nsq
[google_group]: http://groups.google.com/group/nsq-users
[snakes_twitter]: https://twitter.com/imsnakes
[jehiah_twitter]: https://twitter.com/jehiah
[bitly]: https://bitly.com
[contributors]: https://github.com/bitly/nsq/graphs/contributors
[nsqio_twitter]: https://twitter.com/nsqio
