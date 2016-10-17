#Prictise for Pacer

## TCP 和 UDP 的区别 ##

1. TCP面向连接；UDP是无连接的，即发送数据之前不需要建立连接
2. TCP提供可靠服务。也就是说，通过TCP连接传送的数据，无差错，不丢失，不重复，且按序到达；UDP尽最大努力交付，即不保证可靠交付
3. TCP面向字节流，实际上是TCP把数据看成一连串无结构的字节流；UDP是面向报文的，UDP没有拥塞控制，因此网络出现拥塞不会使源主机的发送速率降低(对实时应用很有用，如IP电话，实时视频会议等)
4. 每一条TCP连接只能是点到点的；UDP支持一对一，一对多，多对一和多对多的交互通信
5. TCP首部开销20字节；UDP的首部开销小，只有8个字节
6. TCP的逻辑通信信道是全双工的可靠信道，UDP则是不可靠信道

## 一次完整的HTTP事务是怎样一个过程？ ##

http://blog.csdn.net/yipiankongbai/article/details/25029183


当我们在浏览器的地址栏输入 www.linux178.com ，然后回车，回车这一瞬间到看到页面到底发生了什么呢？

以下过程仅是个人理解：

1. 域名解析
2. 发起TCP的3次握手
3. 建立TCP连接后发起http请求
4. 服务器响应http请求，浏览器得到html代码
5. 浏览器解析html代码，并请求html代码中的资源(如 js, css, 图片等)
6. 浏览器对页面进行渲染呈现给用户


## Http GET和POST的区别##

### GET请求的数据会暴露在地址栏中，POST请求会把请求的数据放置在HTTP请求包的包体中

###传输数据的大小

在HTTP规范中，没有对URL的长度和传输的数据大小进行限制。但是在实际开发过程中，对于GET，特定的浏览器和服务器对URL的长度有限制。因此，在使用GET请求时，传输数据会受到URL长度的限制。

对于POST，由于不是URL传值，理论上是不会受限制的，但是实际上各个服务器会规定对POST提交数据大小进行限制，Apache、IIS都有各自的配置。

###安全性

POST的安全性比GET的高。这里的安全是指真正的安全，而不同于上面GET提到的安全方法中的安全，上面提到的安全仅仅是不修改服务器的数据。比如，在进行登录操作，通过GET请求，用户名和密码都会暴露再URL上，因为登录页面有可能被浏览器缓存以及其他人查看浏览器的历史记录的原因，此时的用户名和密码就很容易被他人拿到了。除此之外，GET请求提交的数据还可能会造成Cross-site request frogery攻击