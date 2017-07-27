# SSH登陆与隧道

<!-- TOC depthFrom:1 depthTo:6 withLinks:1 updateOnSave:1 orderedList:0 -->

- [SSH登陆与隧道](#ssh登陆与隧道)
	- [私钥登陆](#私钥登陆)
	- [用图示意](#用图示意)
	- [生成私匙](#生成私匙)
	- [分发公钥](#分发公钥)
	- [参考资料](#参考资料)

<!-- /TOC -->

## 私钥登陆

日常SSH认证[1]，主要有两种方式：
- 一个是账号密码；
- 另一个是``private key``。

**注意**

>其实并不是 ``public key``，只不过运维给我们开通权限的时候总是问我们提供``public key``，就被大家误以为是``公钥登陆``。实际上认证的时候是用``private key``，用的是签名（签名用``private key``，验签是用``public key``）。

多数大公司SSH登陆方式是：先通过``private key``登陆跳板机，再穿梭到其他主机。

**自动读取私钥**

>当我们输入：``ssh someuser@10.10.1.234`` 时，``ssh``会自动读取``~/.ssh/id_rsa``文件（这个文件就是``private key``），以这个验证身份。

## 用图示意

![](assets/ssh-pri-key.png)

如图有三步，后续文字想详讲它们。

## 生成私匙

为了申请登陆服务器，运维通常会要求我们提供自己的``公钥``。我们需要生成``秘钥对``，然后把``私钥``保密，把``公钥``给运维。
你可能会问：为什么运维不帮你生成``秘钥对``呢？然后把``私钥``给你，你放到``~/.ssh/id_rsa``位置不就行了么？的确是可以的。只不过那样就失去安全意义了。你的``私钥``都交给运维了，那你的账号做了些破坏性操作时，并不能证明操作人就是你。

生成``秘钥对``，用``ssh-keygen``命令就好：

``` bash
$ ssh-keygen

按提示，输入必要信息。最后私钥生成到 ~/.ssh/id_rsa，公钥是 ~/.ssh/id_rsa.pub
```
**注意**
>如果您担心``私钥``直接放在本机的``~/.ssh/id_rsa``文件怕别人看见，您还可以给``私钥``加个口令（就是查看它的时候，需要输入一个口令）。

## 分发公钥

假设有两个Linux主机，分别命名叫C和S，现在要 **从C登陆到S** 上。如何配置？

- 私钥文件： 在C的 ``~/.ssh`` 中要保存“私钥”，文件名为 ``~/.ssh/id_rsa``。
- 分发公钥： 在S的 ``~/.ssh/authorized_keys`` 文件中得追加C的 ``public key``。

 ``` bash
cat id_rsa.pub >> ~/.ssh/authorized_keys
 ```

 >注意：``id_rsa.pub``是C的，``authorized_keys``是S的。

- 更新权限：在S的 ~/.ssh 执行chmod 0600 *

-----


## 参考资料

- [Java SSH 工具包（JSCH） 深入浅出](http://xliangwu.iteye.com/blog/1499764)
- [JSCH Demo 代码](http://xliangwu.iteye.com/blog/1499764)
- [如何配置private key 登陆方式？](http://xliangwu.iteye.com/blog/1499764)
- [免密登陆](http://blog.chinaunix.net/uid-26284395-id-2949145.html): ssh-keygen,ssh-copy-id和authorized_keys
- [如何通过SSH访问mysql ?](http://stackoverflow.com/questions/1968293/connect-to-remote-mysql-database-through-ssh-using-java)
- [跳板机](http://blog.csdn.net/mdl13412/article/details/8986412): 登陆跳板机时，需要动态密码
- [SSH Tunnel 隧道](http://blog.csdn.net/blade2001/article/details/8877250)
- [反向隧道](http://my.oschina.net/abcfy2/blog/177094)
- [SSH 正向/反向隧道](http://blog.csdn.net/quqi99/article/details/7334617)
- [SSH 隧道](http://blog.csdn.net/zenghui08/article/details/7896520)
- [多层 SSH隧道](http://wenku.baidu.com/link?url=8rK3IwF8kH-mV45yU7Edbd9iKc45PJuLzdRF1djWt9TwFkwkX78lYF7kjx0ZWtyI-Bbn8EO4_WD-TKqrTOZb5GsvuYkQ-vnbv9ySegVgfIi)
