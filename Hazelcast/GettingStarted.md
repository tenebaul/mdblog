# Hazelcast

## Hazelcast 初体验

- 类比Redis：``Hazelcast`` 数据结构上就好比是 Java版的 ``Redis``。
- 天然分布式：但比Redis牛逼的是支持分布式。
- 嵌入式：``Hazelcast``可以以 ``Embeded`` 的形式运行于JVM，这样Java应用可以直接本地内存计算，性能大大提升，而且自动分布式。
- 跨语言：Java的程序，比如``ehcache``很大的问题就是只局限于Java。这样的程序走不远，因为现代互联网企业都是多种语言的。为什么谈架构很多都是C的呢？比如``nginx``，再比如``redis``都是C的，但是他们都提供各种语言的客户端，只有这样才能走得更远。``Hazelcast``设计时，除了像``ehcache``那样支持嵌入式，还像``redis``那样，支持C/S结构，从而跨语言了。
- 简洁API：``Hazelcast``当做缓存用的时候，完全遵守``JCache``标准，降低用户使用门槛；而当它当做数据结构服务器的时候，比如``Map``，``Queue``等的时候，它又完全遵守``java.util.Map``里面的接口定义。
- 集群成员自发现：``Hazelcast``的集群管理完全自动化，成员会自发现，包括新增成员和成员宕机。比我们之前用``JGroups``来做成员发现和数据复制要简单很多很多。


## Hazelcast 官方资料

- [创始人讲解Hazelcast入门](https://youtu.be/y6Bc3XOpJQ8)
- [getting-started-with-hazelcast](https://hazelcast.org/getting-started-with-hazelcast/)
- [架构图](http://docs.hazelcast.org/docs/latest/manual/html-single/images/HCArch.png)
