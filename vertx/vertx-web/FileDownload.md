

>you need to create a ``Pump`` between an HttpServerResponse (WriteStream) and an AsyncFile (ReadStream), it will do that for you.



## sendFile

why don’t you simply use

``` java
response.sendFile()
```

## Streams

>In Vert.x, write calls return immediately, and writes are queued internally.  

在 vertx 里，写操作都是立即返回的。写的时候，并不是真的就写出去了，而是在内部排队（也就是仅仅写入了“写队列”）。


>It’s not hard to see that if you write to an object **faster than** it can actually write the data to its **underlying resource**, then the write queue can grow unbounded - eventually resulting in memory exhaustion.

为了感知写队列的存在，只要写操作比底层链路资源实际写操作要快，就会在队列里积累，并导致内存溢出。








# 参考资料

- [Streams](http://vertx.io/docs/vertx-core/java/#streams)
