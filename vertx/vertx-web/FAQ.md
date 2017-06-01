

- [No distinction between Query and Path params #304](https://github.com/vert-x3/vertx-web/issues/304)

路径参数：http://some.host.com/:name
歧义请求：http://some.host.com/fred?name=george

按照路径参数：name=fred；但是按照QueryString，name=george。
那么实际的参数值是多少呢？答：是个数组，因为HTTP协议标准规定QueryString可以多个同名field。

>Once you use both **query string and path params** with the same name you will end up with a **unordered** list with all results for example in your case: [fred, george].


设计者为什么这么设计呢？

>@plenderyou path params are a not official "parameters" in the sense that they are not part of the HTTP query string, so in vert.x-web as **in many other frameworks they are just masked as being part of the query string**.

某个field支持List，但多数情况是标量，所以设计了两个API：

If you want to be sure there's only 1 value then you should use ``rc.request.params.getAll('name')`` and work with the list. If you just use ``get('name')`` then you only get the **last added** value which is the path one (**query processing happens before path when filling the variable**).

有人还是坚持应该区分，理由是：

>I don't consider this a bug, either a choice or a missing feature, but I do agree it'd be **a nice improvement**. Many frameworks provide this distinction (think of JAX-RS @QueryParam vs. @PathParam or Spring MVC ``@PathVariable`` vs. ``@RequestParam``).


path param

``` java
Map<String, String> params = rc.pathParams();
```

普通的: params
``` java
rc.request().params();
```
