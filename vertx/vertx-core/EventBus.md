
## 三种消息模型

Event Bus支持三种消息机制：
- 发布/订阅(Publish/Subscribe)
- 点对点(Point to point)
- 请求/回应(Request-Response)模式。


### Pub/Sub 模式：复制

``` java
EventBus eventBus = vertx.eventBus();

// Pub
eventBus.publish("foo.bar.baz", "message");

// Two Subs: Subscribe to 'foo.bar.baz' address

eventBus.consumer("foo.bar.baz", r -> {
  System.out.println("1: " + r.body());
});

eventBus.consumer("foo.bar.baz", r -> {
  System.out.println("2: " + r.body());
});

```

### P2P 模式：轮训


``` java
EventBus eventBus = vertx.eventBus();

// Pub
eventBus.send("foo.bar.baz", "message");

// Two Subs: Subscribe to 'foo.bar.baz' address

eventBus.consumer("foo.bar.baz", r -> {
  System.out.println("1: " + r.body());
});

eventBus.consumer("foo.bar.baz", r -> {
  System.out.println("2: " + r.body());
});

```

P2P 模式跟Pub/Sub模式，在代码上，唯一的区别是：前者是 ``send``，后者是 ``publish`` 方法。

接收效果上，P2P是只有一个收到消息，如果有两个监听者，它们会被轮询。而Pub/Sub模式下，每个监听者都会收到消息。

**总结**

> ``send``和 ``publish``的逻辑相近，只不过一个是发送至目标地址的**某一**消费者，一个是发布至目标地址的**所有**消费者。

### 请求/回应模式

当我们绑定的Handler接收到消息的时候，我们可不可以给消息的发送者回复呢？当然了！当我们通过send方法发送消息的时候，我们可以同时指定一个回复处理函数(reply handler)。然后当某个消息的订阅者接收到消息的时候，它就可以给发送者回复消息；如果发送者接收到了回复，发送者绑定的回复处理函数就会被调用。这就是请求/回应模式。


# 参考资料

- [Vertx Eventbus not working in Java](https://stackoverflow.com/questions/30983863/vertx-eventbus-not-working-in-java)

- [关于Java框架Vert.x的几点思考](http://www.csdn.net/article/2015-05-20/2824733-Java)
