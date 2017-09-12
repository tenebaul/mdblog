# PUT Objects

## 请求初接触

``` http
PUT /ObjectName HTTP/1.1
Host: BucketName.s3.amazonaws.com
Date: date
Authorization: authorization string
```


## 请求详解

The following request stores the image ``my-image.jpg`` in the bucket ``myBucket``.

``` http
PUT /my-image.jpg HTTP/1.1
Host: myBucket.s3.amazonaws.com
Date: Wed, 12 Oct 2009 17:50:00 GMT
Authorization: authorization string
Content-Type: text/plain
Content-Length: 11434
x-amz-meta-author: Janet    （自定义meta头：author=Janet）
Expect: 100-continue
[11434 bytes of object data]
```

### 请求头列表

Request headers are limited to 8 KB in size

- ``Content-MD5``: The base64 encoded 128-bit MD5 digest of the message (without the headers) according to RFC 1864. This header can be used as a ``message integrity check``（消息完整性校验） to verify that the data is the same data that was originally sent. Although it is **optional**, we recommend using the Content-MD5 mechanism as an end-to-end integrity check.

- ``Cache-Control``: 当读取文件的时候，是否采用缓存策略。
- ``Content-Disposition``: 当读取文件的时候，下载时，文件名是否支持用户重新定义。注意：文件名与文件ID是分离的。
- ``Content-Type``: 文件类型，比如文本、图片还是视频。A standard ``MIME`` type describing the format of the contents.

- ``x-amz-meta-``： 自定义``meta``信息。Headers starting with this prefix are ``user-defined`` metadata（注意那个只是前缀）. Within the PUT request header, the user-defined metadata is **limited to 2 KB in size**. User-defined metadata is **a set of key-value pairs**. The size of user-defined metadata is measured by taking the sum of the number of bytes in the UTF-8 encoding of each key and value. Amazon S3 doesn't validate or interpret user-defined metadata.

### 特色头

- ``x-amz-tagging``：尽管``S3``不支持文件系统的``POSIX``接口。但是它在KV的结构上，支持``tag``检索。这非常适合图片社交网站，每个图片会打上``tag``，然后可以按``tag``来归类。Specifies a set of one or more tags you want to associated with the object.

## 优秀的设计

### 安全性校验

``` http
PUT /ObjectName HTTP/1.1
Host: BucketName.s3.amazonaws.com
Date: date
Authorization: authorization string
```

其中``Authorization: authorization string``，表示签名。对``Bucket``的操作，需要用Key做签名。而对对象的操作，还有ACL控制。

### 先验证，再上传

``Amazon``就是``Amazon``，它考虑问题多么细致呢？！我们上传一个300MB的文件，花了不少时间，好不容易上传完，结果服务端提示签名校验失败。这是什么心情？难道权限校验不能提前校验吗？

To configure your application to send the request headers **prior to** sending the request body, use the 100-continue HTTP status code.

For PUT operations, this helps you **avoid sending the message body if the message is rejected based on the headers** (e.g., because of authentication failure or redirect). For more information on the 100-continue HTTP status code, go to Section 8.2.3 of http://www.ietf.org/rfc/rfc2616.txt.


- 先上传头，再上传包体

``` http
PUT /my-image.jpg HTTP/1.1
Host: myBucket.s3.amazonaws.com
Date: Wed, 12 Oct 2009 17:50:00 GMT
Authorization: authorization string
Content-Type: text/plain
Content-Length: 11434
x-amz-meta-author: Janet
Expect: 100-continue   （注意：包体部分，会等待服务端响应 100-continue 后，才会正式开始上传）
[11434 bytes of object data]
```

- 服务端响应：先响应100-continue，再响应正式数据

``` http
HTTP/1.1 100 Continue

HTTP/1.1 200 OK
x-amz-id-2: LriYPLdmOdAiIfgSm/F1YsViT1LW94/xUQxMsF7xiEb1a0wiIOIxl+zbwZ163pt7
x-amz-request-id: 0A49CE4060975EAC
Date: Wed, 12 Oct 2009 17:50:00 GMT
ETag: "1b2cf535f27731c974343645a3985328"
Content-Length: 0
Connection: close
Server: AmazonS3
```


### 并发控制

当多个客户端并发向一个``Object``发起``PUT``操作，会表现为覆盖操作。也就是数据库领域常说的``Lost Update``。那客户端会比较纳闷，它刚写完后，读一下，却发现是别的东西。``S3``没有采用悲观锁来禁止并发，但采用了乐观锁（返回版本信息）让客户端来比对。

>Amazon S3 is a distributed system. If it receives multiple write requests for the same object simultaneously, **it overwrites all but the last object written**. Amazon S3 **does not provide object locking**; if you need this, make sure to build it into your application layer or use **versioning** instead.

### 完整性校验

>To ensure that data is not corrupted traversing the network, use the ``Content-MD5`` header. When you use this header, Amazon S3 checks the object against the provided MD5 value and, if they do not match, returns an error. **Additionally**, you can calculate the MD5 while putting an object to Amazon S3 and compare the returned ETag to the calculated MD5 value.

当我们上传一个文件的时候，怎么保证我们上传的东西到服务器没有被篡改呢？办法是端到端比对``MD5``。客户端可以计算文件的MD5，然后把文件上传到服务端，并在HEAD中携带MD5，服务器收到后，也计算MD5，并跟客户端声明的比对，如果吻合，则表示没有篡改。

如果你觉得上传前，计算MD5比较浪费时间。你也可以先上传，并在上传过程中计算MD5，这样上传和计算MD5是并行进行的。等服务端返回``ETag``（也是MD5，只不过是服务端计算的），客户端可以把自己计算的MD5跟``ETag``比对。

>我想问的是，能不能先上传文件，并在稍后上传HTTP HEAD呢？答案：不可以。HTTP协议不支持HEAD后面传。在``trunk-transfer``中，可以支持这种流式的场景。

### 数据存储安全

对于云存储我们非常担心数据安全问题。但是前面讲的这些都是在防止传输层的安全问题，比如签名呀，MD5校验呀。但是我们就信任amazon吗？或者amazon某些不遵守职业操守的员工呢？为此，amazon专门提供数据存储加密机制，而且这个加密秘钥可以上传用户自定义的。

You can optionally request **server-side encryption** where Amazon S3 **encrypts your data as it writes it to disks** in its data centers and decrypts it for you when you access it.

You have the option to provide your own **encryption key** or use AWS-managed encryption keys. For more information, go to Using Server-Side Encryption in the Amazon Simple Storage Service Developer Guide.



## 参考资料

- [Object PUT](http://docs.aws.amazon.com/AmazonS3/latest/API/RESTObjectPUT.html)
- [Common Request Headers](http://docs.aws.amazon.com/AmazonS3/latest/API/RESTCommonRequestHeaders.html)
