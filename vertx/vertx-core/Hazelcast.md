# 缓存 与 Hazelcast

我们为什么要用缓存？
太简单的问题，因为要提高速度~ 中心思想是空间换时间，从数据库中预加载一部份数据放到内存或磁盘中，牺牲空间来换取交互时间。

一般的应用正式环境中都不止一台服务器（也就是说是集群的），那么如果只是简单的将数据预加载到内存，那么就会有数据不同步的现象。（更新了其中一台JVM，另一台JVM并不会收到通知从而保持数据同步）。这时候就需要用到cache server了。


目前流行的cache server有很多种：
- memcache
- redis
- ehcache
- Hazelcast

相比其它三种，``Hazelcast`` 好像并没有那么流行，中文文档比较少。


## Hazelcast

The Leading ``Open Source`` In-Memory ``Data Grid``

Distributed Computing, Simplified

In-Memory Data Grid ⋅ JCache Provider ⋅ Apache 2 License

Small jar with minimal dependencies ⋅ ``Embedded`` or ``Client Server``

简单说，什么是 ``Hazelcast`` 呢？

>``Hazelcast`` 就是 Redis 的 Java 版本。

## Apache Ignite

Apache Ignite is a high-performance, integrated and distributed in-memory platform for computing and transacting on large-scale data sets in real-time, orders of magnitude faster than possible with traditional disk-based or flash technologies.

>Apache Ignite是一个通用的数据库缓存系统，它不仅支持所有的底层数据库系统，比如RDBMS、NoSQL和HDFS，还支持Write-Through和Read-Through、Write-Behind Caching等可选功能。

- **Runs Everywhere**

Ignite writes objects to cache in a common binary format allowing applications to interoperate between Java, .Net, and C++.

将数据存储在缓存中能够显著地提高应用的速度，因为缓存能够降低数据在应用和数据库中的传输频率。Apache Ignite允许用户将常用的热数据储存在内存中，它支持**分片和复制**两种方式，让开发者可以均匀地将数据分布式到整个集群的主机上。同时，Ignite还支撑任何底层存储平台，不管是RDBMS、NoSQL，又或是HDFS。


### Write-Through和 Read-Through

- ``Write-Through``: 在Write-Through模式中，缓存中的数据更新会被同步更新到数据库中。

- `` Read-Through``:  Read-Through则是指请求的数据在缓存中**不可用**时，会自动从数据库中拉取。

### Write-Behind Caching

Ignite还提供了一种叫做Write-Behind Caching的数据库异步更新模式。默认情况下，Write-Through中每一次更新都会对数据库发起一次请求。

如果使用 ``Write-Behind Caching`` 后写，对缓存的更新会整合成批次然后再发送给数据库。这对改删频繁的应用来说可以达到相当的性能提升。


### 自动化持久数据

Ignite提供了易用的schema映射工具，从而系统可以自动地与数据库整合。这一工具可以自动地连接数据库，并生成所有需要的XML OR-mapping配置以及Java域模型POJOs。


### SQL查询

查询Ignite缓存很简单，使用的就是标准的SQL。

尽管是 ``Key/Value`` 的存储，但是却用 SQL API 。
