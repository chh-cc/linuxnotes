# https

## 对称加密和非对称加密

举个例子：班花传纸条给我，别人可以看到纸条的内容，于是班花提前跟我说字母颠倒过来就是想跟我说的内容，这样就只有我能看懂了。

将一段明文转变为密文，就是**加密**，反之就是**解密**。像这种颠倒过来的加密方法，其实就是**秘钥**，而这种加密和解密都是用同一种秘钥的加密形式，就叫**对称加密**。

而**非对称加密**是加密和解密用的是两个不同的秘钥，分别是**公钥和私钥**，公钥负责加密，私钥负责解密，用公钥加密过的密文只有私钥才能解密，反过来说私钥加密过的密文只有公钥才能解密也是ok的，这种操作就是常说的**验证数字签名**。

## https的加密原理

如果代码只对内网提供服务，那大概率服务用的是http，如果要将服务暴露到公网，那还用HTTP的话，信息收发就是明文，只要有人在通讯链路中的任意一个路由器抓个包，就能看到http包里的内容。

为了让明文变为密文，需要在**http层之上再加一层TLS层**，目的是为了加密，这就是常说的https。

## https连接怎么建立

首先是建立TCP三次握手，然后就可以进入https的加密流程

加密流程分为两个阶段：TLS四次握手和加密通信



HTTPS既用到了非对称加密（TLS四次握手），也用到了对称加密。，原因是对称加密相对更快。