

## 基础

### 编译源码

官方提供了源码和 [二进制版](http://www-eu.apache.org/dist//ignite/2.0.0/apache-ignite-fabric-2.0.0-bin.zip)


从 https://ignite.apache.org/ 下载Apache Ignite的zip压缩包

``` bash

# Unpack the source package
$ unzip -q apache-ignite-{version}-src.zip
$ cd apache-ignite-{version}-src


# Build In-Memory Data Fabric release (without LGPL dependencies)
$ mvn clean package -DskipTests

# Build In-Memory Data Fabric release (with LGPL dependencies)
$ mvn clean package -DskipTests -Prelease,lgpl


# Build In-Memory Hadoop Accelerator release
# (optionally specify version of hadoop to use)
$ mvn clean package -DskipTests -Dignite.edition=hadoop [-Dhadoop.version=X.X.X]

```

编译有很多可选功能。基础的是 ``Data Fabric``, 其他的可以有 ``Hadoop Accelerator`` 功能。

### 从命令行启动

- 默认配置文件：

``` bash
$ bin/ignite.sh

```

- 指定配置文件：

``` bash
$ bin/ignite.sh config/default-config.xml
```

### 从maven获得

Ignite只需要一个``ignite-core``强依赖，通常还需要添加``ignite-spring``，来做基于spring的XML配置，还有``ignite-indexing``,来做SQL查询。

ignite.version = 2.0.0

``` xml
<dependency>
    <groupId>org.apache.ignite</groupId>
    <artifactId>ignite-core</artifactId>
    <version>${ignite.version}</version>
</dependency>
<dependency>
    <groupId>org.apache.ignite</groupId>
    <artifactId>ignite-spring</artifactId>
    <version>${ignite.version}</version>
</dependency>
<dependency>
    <groupId>org.apache.ignite</groupId>
    <artifactId>ignite-indexing</artifactId>
    <version>${ignite.version}</version>
</dependency>
```

### 集群成员自发现

>启动很多很多的节点然后他们会自动地发现对方。


## 其他模块

- ignite-spring：基于Spring的配置支持
- ignite-indexing：SQL查询和索引
- ignite-geospatial：地理位置索引
- ignite-hibernate：Hibernate集成
- ignite-web：Web Session集群化
- ignite-schedule：基于Cron的计划任务
- ignite-log4j：Log4j日志
- ignite-jcl：Apache Commons logging日志
- ignite-jta：XA集成
- ignite-hadoop2-integration：HDFS2.0集成
- ignite-rest-http：HTTP REST请求
- ignite-scalar：Ignite Scalar API
- ignite-slf4j：SLF4J日志
- ignite-ssh；SSH支持，远程机器上启动网格节点
- ignite-urideploy：基于URI的部署
- ``ignite-aws``：AWS S3上的无缝集群发现
- ignite-aop：网格支持AOP
- ``ignite-visor-console``：开源的命令行管理和监控工具

## 生命周期


Ignite是基于JVM的，一个JVM可以运行一个或者多个逻辑Ignite节点（大多数情况下，一个JVM运行一个Ignite节点）。

>Ignite运行时 == JVM进程 == Ignite节点（多数情况下）

### 启动一个节点

- 默认配置文件

``` java
Ignite ignite = Ignition.start();
```

- 指定配置文件

``` java
Ignite ignite = Ignition.start("examples/config/example-cache.xml");
```

注意：相对路径不是相对当前工作目录，而是相对 ``$IGNITE_HOME`` 的。

### LifecycleBean

有时可能希望在Ignite节点启动和停止的之前和之后执行特定的操作。

``` java
// Create new configuration.
IgniteConfiguration cfg = new IgniteConfiguration();

// Provide lifecycle bean to configuration.
cfg.setLifecycleBeans(new MyLifecycleBean());

// Start Ignite node with given configuration.
Ignite ignite = Ignition.start(cfg)
```


BEFORE_NODE_START：Ignite节点的启动程序初始化之前调用
AFTER_NODE_START：Ignite节点启动之后调用
BEFORE_NODE_STOP：Ignite节点的停止程序初始化之前调用
AFTER_NODE_STOP：Ignite节点停止之后调用


### 异步调用

``` java

// Synchronous get
V get(K key);

// Asynchronous get
IgniteFuture<V> getAsync(K key);
```

## 客户端和服务端

Embed模式 和 C/S模式：

- Redis 这种是典型的C/S模式
- EhCache 是 Embed 模式

``Ignite`` 同时支持Embed和C/S模式。

>所有的Ignite节点默认都是**以服务端模式启动**的，客户端模式需要显式地启用。

``` java
IgniteConfiguration cfg = new IgniteConfiguration();

// Enable client mode.
cfg.setClientMode(true);

// Start Ignite in client mode.
Ignite ignite = Ignition.start(cfg);
```

>服务端节点参与缓存、计算执行、流式处理等等，而原生的客户端节点提供了远程连接服务端的能力。

x\
