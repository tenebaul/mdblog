
# 文件上传与mutli-part编码

<!-- TOC depthFrom:1 depthTo:6 withLinks:1 updateOnSave:1 orderedList:0 -->

- [文件上传与mutli-part编码](#文件上传与mutli-part编码)
	- [提交表单](#提交表单)
	- [提交JSON](#提交json)
- [-d to send raw data](#-d-to-send-raw-data)
	- [提交XML](#提交xml)
	- [上传文件](#上传文件)
		- [只包含单个文件](#只包含单个文件)
		- [单个文件和meta](#单个文件和meta)
		- [多个文件及其meta](#多个文件及其meta)
		- [cURL 帮助](#curl-帮助)
	- [参考资料](#参考资料)

<!-- /TOC -->

## 提交表单

POSTing Form Data with cURL

``` bash
curl -X POST -F 'username=davidwalsh' -F 'password=something' http://domain.tld/post-to-me.php
```

在服务端，如果您使用PHP，那么可以直接从``multi-part`` Form表单中提取参数：

``` PHP
Array(
  'username' => 'davidwalsh',
  'password' => 'something'
)
```

## 提交JSON

POSTing JSON Data with cURL

``` bash
# -d to send raw data
curl -X POST -H 'Content-Type: application/json' -d '{"username":"davidwalsh","password":"something"}' http://domain.tld/login
```

## 提交XML

POSTing XML Data from local file with cURL

``` bash
curl -i -X POST host:port/post-file -H "Content-Type: text/xml" --data-binary "@path/to/file"
```

**注意**

- ``--data-binary``： 表示原始二进制数据格式。
- ``@XXX``: 表示文件，不是一个普通的Value。

## 上传文件

POSTing Files with cURL

### 只包含单个文件

POSTing a file with cURL is slightly different in that you need to add an ``@`` before the file location, after the field name:

``` bash
curl -X POST -F 'image=@/path/to/pictures/picture.jpg' http://domain.tld/upload
```

注意：这里的``image``字段名，可以随便换成别的。那是不是意味着这个参数没有意义呢？并不是的，当我们一条命令，上传多个文件的时候，它用来给多个文件做标识符。

如果服务端用PHP，那么您可以如下方式提取文件和它的meta信息：

``` PHP
Array(
  "image": array(
    "name" => "picture.jpg"
    "type" => "image/jpeg",
    "tmp_name" => "/path/on/server/to/tmp/phprj5rkG",
    "error" => 0,
    "size" => 174476
  )
)
```

### 单个文件和meta

``` bash
curl -F "upF1=@thrift-0.9.3.tar.gz;" -F "nameF1=thrift"  -F "useF1=123456" http://127.0.0.1:10001/files
{"thrift-0.9.3.tar.gz":{"size":8897936}}%
```

这个请求，除了在``upF1``参数上，弄了一个``@``文件；其他两个参数``useF1``和``nameF1`都是关于这个文件的meta描述信息。

### 多个文件及其meta

``` bash
$ curl -i -F  "upF1=@thrift-0.9.3.tar.gz;" -F "nameF1=thrift"  -F "useF1=123456" -F "upF2=@test.db" -F "nameF2=test" -F "userF2=654321" http://127.0.0.1:10001/files

HTTP/1.1 200 OK
Server: boxgate
Content-Type: application/json;charset=UTF-8
Content-Length: 65

{"thrift-0.9.3.tar.gz":{"size":8897936},"test.db":{"size":16384}}%
```

上述一条命令上传了两个文件，一个是``thrift-0.9.3.tar.gz``，另一个是``test.db``，它们的meta信息，通过``multi-part``编码一起传送，分别是``nameFi``和``useFi``，其中``i``表示1，2，…,N，表示第i个文件。

这种方式虽然可以，也很灵活。但是为什么对File的meta描述没有标准化呢？

### cURL 帮助


>-F/--form <name=content>

>(HTTP) This lets curl emulate a filled-in form in which a user has pressed the submit button. This causes curl to POST data using the Content-Type ``multipart/form-data`` according to ``RFC2388``. This enables uploading of binary files etc. To force the 'content' part to be a file, prefix the file name with an @ sign.

---

## 参考资料

- [post file using curl command line](https://davidwalsh.name/curl-post-file)

- [Using curl to upload POST data with files](https://stackoverflow.com/questions/12667797/using-curl-to-upload-post-data-with-files)
