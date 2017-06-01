
# Reroute doesn't pass query string


 [#405](https://github.com/vert-x3/vertx-web/issues/405)

- 提出问题

用户 [comradesharf](https://github.com/comradesharf) 在2016-7-1日，报告了一个 ISSUE， 说 vertx 从 3.3.0 版本后，reroute （类似nginx的url-rewrite）时，只传递 HTTP方法和路径，但是 QueryString 被干掉了。

>After this pull request [#355](https://github.com/vert-x3/vertx-web/pull/355) parameters from query string are **not passed anymore on reroute**.  ``Vertx Version`` 3.3.0

- 解决问题

紧接着当天有个 vertx 的 member , 叫 [pmlopes](https://github.com/pmlopes)， 说这个问题已经描述了。

>That was documented here: http://vertx.io/docs/vertx-web/java/#_reroute

> On reroute the request is "reset" and it starts over, this means that path params are cleared and **if new ones are provided they will be parsed**.

- 不同意见

尽管member把这个问题关闭了，因为他认为``reroute``的语义就是``reset``，所以需要把参数都干掉。但是其他网友认为决定权应该交给用户，它是一个可选的行为。

>Hi @pmlopes , I also have a question to this topic. Do we need to **enforce rerouting based on path only**? Why wouldn't we reroute based on (relative) uri, so that we can reroute a request and preserve its query parameters?

>```
routingCtx.reroute("/anotherPath?" + routingCtx.request().query());
```


----

# 附录-1: Reroute


>Until now all routing mechanism allow you to handle your requests **in a sequential way**, however there might be times where you will want to go back.

vertx routing 算法非常简单，远远比Nginx的 location matching 简单多了，vertx 遵循``职责链``模式，本质上就是**顺序检索**: 当一个请求过来后，逐个查看是否匹配，如果不匹配，直接路过，接着看下一个节点，如果匹配，则执行节点处理，同时由程序员决定是否再往后看其他节点。

这里的``reroute``，就是nginx的``url rewrite``机制。rewrite后的HTTP请求，会被重新进行Location的匹配。



>You can reroute based on a new path or based on a new path and method. Note however that rerouting based on method might introduce security issues since for example a usually safe GET request can become a DELETE.

``Reroute``不仅可以重写路径，还可以重写方法。比如 ``GET /files``， 可以内部重写为 ``DELETE /path/to/file``。这完全取决于程序员，尽管这种风格并不友好。
