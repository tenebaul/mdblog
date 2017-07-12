# Discovery Vertx

## 参考资料

- [vertx-service-discovery](http://vertx.io/docs/vertx-service-discovery/java/)

## 快速简介

### 什么是``Service``？

A service is a ``discoverable`` functionality.
It can be **qualified by** its type, metadata, and location.

>这个概念就类似``AppStore``，只不过``AppStore``管理的对象是``App``，发挥的作用是连接``App Provider``与``App consumer``。而``Service Discovery``管理对象是``Service``（是服务API），发挥作用是一样的。当然除了``Service Discovery``之外，更重要的是处理``LB``和``HA``的问题。

``Service``是什么呢？不仅仅是``RESTful API``，它可以是一切资源。

>So a ``service`` can be a **database**, a service proxy, a **HTTP endpoint** and any other resource you can imagine as soon as you can describe it, discover it and interact with it.


>Each service is described by a ``Record``. the central piece of information shared by the providers and consumers are ``records``.

### Service Provider

A service provider can:
- publish a service record
- un-publish a published record
- update the ``status`` of a published service (down, out of service…​)

### Service Consumer

A service consumer can:
- lookup services
- bind to a selected service (it gets a ``ServiceReference``) and use it
- release the service once the consumer is done with it
- **listen for arrival, departure and modification of services** .


>Consumer would
- 1) lookup a service record matching their need,
- 2) retrieve the ``ServiceReference`` that give access to the service,
- 3) get a service object to access the service,
- 4) release the service object once done.


## 术语定义

### Service records

>A service ``Record`` is an object that **describes a service** published by a service provider.

It contains a name, some metadata, **a location object** (describing where is the service).
This record is the only object shared by the provider (having published it) and the consumer (retrieve it when doing a lookup).

A record is published when the provider is ready to be used, and withdrawn when the service provider is stopping.


>服务描述本身并不容易，比如``RESTful API``的描述，就有``Swagger``标准。

### Service Provider and publisher

A service provider is an entity providing a service. The publisher is responsible for publishing a record describing the provider. It may be a single entity (a provider publishing itself) or a different entity.

### Service Consumer

>Service consumers search for services in the service discovery. Each lookup retrieves ``0..n Record``。
>From these records, a consumer can retrieve a ``ServiceReference``, representing the binding between the consumer and the provider. **This reference allows the consumer to retrieve the service object (to use the service), and release the service**. It is important to release service references to cleanup the objects and update the service usages.

服务发现过程很简单: 客户端向中心查询，中心返回0个或多个``Service Record``。这些``Service Record``被封装成了一个叫``ServiceReference``的东西。这个东西居然是一种``授权码``，客户端拿着这个``授权码``才能访问``Service Provider``？


### Service object

The service object is the object that gives access to a service.
It can come in various forms, such as a proxy, a client, and may even be non-existent for some service types. The nature of the service object depends on the service type.

### Service types
### Service events
### Backend
The service discovery uses a Vert.x ``distributed data structure`` to store the records. So, all members of the cluster have access to all the records.

You can implement your own by implementing the ``ServiceDiscoveryBackend`` SPI. For instance, we provide an implementation based on Redis.

---

# 实验

- ``ServiceDiscoveryPublisher.java``
- ``ServiceDiscoveryConsumer.java``

把一个``RESTful API``发布到服务注册中心：
``http://localhost:10001/dbapi/default/device/``

把进程叫服务，把路径叫服务的meta。于是服务描述为：

``` json
Record record_boxgate = HttpEndpoint.createRecord("boxgate-device",     "localhost", 10001, "/",
				new JsonObject().put("device", new JsonObject().put("listing", "/dbapi/default/device/")));
```


## Pub端

发布一个服务：

``` java
ServiceDiscovery discovery = ServiceDiscovery.create(vertx,
				new ServiceDiscoveryOptions().setAnnounceAddress("service-announce").setName("my-name"));

// create a record from type
Record record_boxgate = HttpEndpoint.createRecord("boxgate-device", "localhost", 10001, "/",
				new JsonObject().put("device", new JsonObject().put("listing", "/dbapi/default/device/")));

// publish the service
discovery.publish(record_boxgate, ar -> {
			if (ar.succeeded()) {

				Record record = ar.result();
				System.out.println("service publish ok: " +
						String.format("name: %s, type: %s, status: %s, "
						+ "registration: %s, location: %s, metadata: %s",
						record.getName(), record.getType(), record.getStatus(),
						record.getRegistration(), record.getLocation(), record.getMetadata()));

			} else {
				System.out.println("service publish fail: " + ar.cause());
			}
		});
```

## Sub端

服务消费：

``` java
discovery.getRecord(r -> r.getName().equals(serviceName), ar -> {
  if (ar.succeeded()) {
    if (ar.result() != null) {

      Record record = ar.result();
      System.out.println("discovery get record: " + String.format(
          "name: %s, type: %s, status: %s, " + "registration: %s, location: %s, metadata: %s",
          record.getName(), record.getType(), record.getStatus(), record.getRegistration(),
          record.getLocation(), record.getMetadata()));

      // Retrieve the service reference
      ServiceReference reference = discovery.getReference(record);

      // Retrieve the service object
      HttpClient client = reference.getAs(HttpClient.class);

      // new JsonObject().put("device", new JsonObject().put("listing", "/dbapi/default/device/"))

      client.getNow(record.getMetadata().getJsonObject("device").getString("listing"), response -> {

        response.bodyHandler(buffer -> {
          System.out.println("device listing Got Response: " + buffer.toString());
        });

        // release the service
        reference.release();

      });
    }
  }

});
```

## 作用

无非是想发挥：``LB``和``HA``的作用。

## 评价

``vertx``的微服务设计真心太一般了。
