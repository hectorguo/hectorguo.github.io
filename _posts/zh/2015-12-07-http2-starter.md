---
layout: post
title: 谈谈HTTP/2对前端的影响
categories: ['zh']
tags: ['前端开发']
published: True
cover: 'https://raw.githubusercontent.com/hectorguo/blog-imgs/master/img/20190608235529.png'
---

随着 HTTP/2 规范的确认，以及主流浏览器（Chrome、Firefox、IE11）对其的全面支持，是时候采用新协议了。看了很多博文跟官方说明，在此做个总结，

* TOC
{:toc}

## 为什么要有 HTTP/2 ？

一句话评价：**预先加载，合并请求，缩小数据，提升性能**。

HTTP 1.1时代，每个TCP连接一次只能下载一个资源，比如浏览器发送一个请求获取`index.html`的数据，服务端收到后只会返回`index.html`，随后浏览器会解析`index.html`，如果里面含有`<link rel="stylesheet" href="xxx.css">`或`<script src="app.js"></script>`，则会再次向服务器发送请求，获取`xxx.css`和`app.js`的数据。 这一过程会产生 **三个性能问题**：

1. 如果`index.html`里含有多个js和css文件，请求数则随之增加，从而导致在TCP往返连接所耗费的时间增多。
2. 每次发送的请求，HTTP头部信息基本是一样的，从而导致一定的头部信息冗余，耗费了不必要的流量。
3. `index.html`与内部的资源文件之间会产生了一个延时，而非同步获取。

由于上述问题，也就催生出了 HTTP/2。在HTTP/2中，多个请求是可以合并为一个的，如下图，多个数据请求允许在同一路中传输（[Multiplexed](https://http2.github.io/faq/#why-is-http2-multiplexed)），这样也就可以解决了 问题1。

![](https://raw.githubusercontent.com/hectorguo/blog-imgs/master/img/20190608235540.png)

而因为是同一个请求，因此 HTTP头信息 只需要有一个就足够，下图可看出，HTTP/2中一个请求头中允许有多个方法，既可以GET，也可以PUT，在切换到下一个方法时，只需要获取数据即可，而不用再次获取头信息。

![](https://raw.githubusercontent.com/hectorguo/blog-imgs/master/img/20190608235548.png)

而且，HTTP/2将头信息进行了压缩（参见[HPACK](https://http2.github.io/faq/#why-hpack)），进一步的减少了头信息的大小，因此 问题2 得到了解决。

同时，HTTP/2 新加入了 `PUSH` 方法，该方法的主要作用就是让服务器试探性的去推送信息给客户端，如 问题3 中所述情况，当请求`index.html`时，服务器在返回`index.html`的同时，会主动把`xxx.css`和`app.js`一同发送给浏览器。这样当浏览器解析DOM，准备发送请求获取`xxx.css`和`app.js`的时候，也许两个资源已经下载完了，只需要从缓存中获取即可。这样就大大减少了网络请求的时间。

![](https://raw.githubusercontent.com/hectorguo/blog-imgs/master/img/20190608235603.png)

## HTTP/2给前端带来哪些影响？

虽然HTTP/2是在协议方面的改进，且但其机制也对目前前端的部分优化方案产生一定影响。

### 减少HTTP请求不一定提升性能

如上所述， HTTP/2 针对多个请求进行了优化，因此之前我们在前端中所做的 **关于减少HTTP请求的最佳实践都不再适用**，如合并JS、CSS文件（[Concatenation](https://hacks.mozilla.org/2012/12/fantastic-front-end-performance-part-1-concatenate-compress-cache-a-node-js-holiday-season-part-4/)），多个图片或图标合并（[Spriting](https://en.wikipedia.org/wiki/Sprite_(computer_graphics)#Sprites_by_CSS)）,将较小的JS或CSS文件内嵌到HTML中（Inlining），合并HTML文件（[Vulcanize](https://github.com/polymer/vulcanize)），根据 [此网站](https://http2-push.appspot.com) 的测试结果显示，在使用HTTP/2后，合并为一个大文件的加载时间反而会比不合并更长。

![](https://raw.githubusercontent.com/hectorguo/blog-imgs/master/img/20190608235639.png)

如上图所示，其中TTFB时间明显减少，所谓[TTFB（Time To First Byte）](https://en.wikipedia.org/wiki/Time_To_First_Byte)，即从浏览器发送请求开始，到接受到来自服务器的返回的第一字节信息（HTTP头信息）结束，之间所耗费的时间。这里就会包含 TCP连接往返（round trip）＋服务器处理时间（如SQL执行）。 因为浏览器在第一次发送请求后，服务器已经预先把其他资源文件一同推送给了浏览器，因此后续的资源请求中，TTFB的时间得到了缩小。

### 压缩仍然需要

有些可能会问，那是不是用了 HTTP/2 后，资源文件也不需要压缩了呢？ 答案是No，压缩文件还是有必要的，毕竟获取一个小文件的时间比大文件更短，HTTP/2并不会帮你自动压缩文件。

可以看出，HTTP/2 目的之一，就在于想把开发环境与生产环境的部署尽量保持一致，减少因为打包合并而产生一些不必要的麻烦。

## 如何使用 HTTP/2 ?

目前一部分主流网站已经采用了 HTTP/2，而很多语言也已经有实施方案，详细的可见该 [列表](https://github.com/http2/http2-spec/wiki/Implementations)， 本文只介绍下如何用 NodeJS 搭建 HTTP/2 站点。

### 创建证书

虽然目前规范中并没有达成一致意见来决定 HTTP/2 是否需要加密（ [Encryption](https://http2.github.io/faq/#does-http2-require-encryption) ），但目前主流的实施方案都是需要加密的（如SSL/TLS），因此，如同 HTTPS，HTTP/2 同样需要创建公私密钥，来搭建站点。

创建Key：

    openssl req -new -newkey rsa:2048 -nodes -keyout localhost.key -out localhost.csr

在输入完身份信息后，输出证书：

    openssl x509 -req -days 365 -in localhost.csr -signkey localhost.key -out localhost.crt

### 搭建服务

如果是初次使用npm安装，则初始化npm依赖管理：

    npm init

此时则会在当前目录中，生成一个`package.json`文件。

安装http2包:

    npm install http2 –save

新建`app.js`文件:

{% highlight javascript %}
var https = require('http2');
var fs = require('fs');

var options = {
  key: fs.readFileSync('localhost.key'),
  cert: fs.readFileSync('localhost.crt')
};

https.createServer(options, function(request, response) {
  fs.readFile(__dirname + request.url, function (err,data) {
    if (err) {
      response.writeHead(404);
      response.end(JSON.stringify(err));
      return;
    }
    response.writeHead(200);
    response.end(data);
  });
}).listen(8080);
{% endhighlight %}

### 验证

这样就搭建完毕，运行即可看到结果

    node app.js

![](https://raw.githubusercontent.com/hectorguo/blog-imgs/master/img/20190608235655.png)

如果想确认是否真的采用了 HTTP/2， 只需要在Chrome Dev Tools > Network中，新增 Protocol，即可看到结果：

![](https://raw.githubusercontent.com/hectorguo/blog-imgs/master/img/20190608235703.png)


2015-12-25 更新：

另外注意一点，如果你是从http转向https，需要在服务端/客户端做个[重定向](http://stackoverflow.com/questions/22453782/nodejs-http-and-https-over-same-port/23975955#23975955)，保证用户访问的站点是https而不是http。

{% highlight javascript %}
// Redirect from http port 80 to https
var http = require('http');
http.createServer(function (req, res) {
    res.writeHead(301, { "Location": "https://" + req.headers['host'] + req.url });
    res.end();
}).listen(80);
{% endhighlight %}

或者在页面端：

{% highlight javascript %}
var host = "YOURDOMAIN.github.io";
if ((host == window.location.host) && (window.location.protocol != "https:"))
    window.location.protocol = "https";
{% endhighlight %}

而且一旦搭建了https，页面内部所有的外部资源都必须实施https，否则会浏览器阻拦：

```
Mixed Content: The page at 'https://localhost:8080/index.html' was loaded over HTTPS, but requested an insecure script 'http://hectorguo.com/Universities-in-US/app.min.js'. This request has been blocked; the content must be served over HTTPS.
```

### 如何使用 PUSH ？

PUSH 是 HTTP/2 的新方法，虽然可以让服务端预先推送资源给客户端，但并不是说只要使用该方法性能就会提升，有时反而会下降。目前官方还未公布最佳实践方法，因此并不建议在生产环境中运用，但可以在开发环境下体验。

Google提供了一个 [SimpleHttp2Server](https://github.com/GoogleChrome/simplehttp2server) 来简单搭建 HTTP/2 并使用 PUSH 方法，NPM中也提供了一个 [http2-push-manifest](https://www.npmjs.com/package/http2-push-manifest) 包，可以自动检测 html文件中的请求资源，方便服务端识别需要PUSH的文件。

## HTTP/2 部署现状

### 客户端

目前各大浏览器对 HTTP/2 的支持度如下：

![](https://raw.githubusercontent.com/hectorguo/blog-imgs/master/img/20190608235713.png)

可以看出基本可以不用担心浏览器的支持度问题，而且由于与HTTP 1.1的API一致，只要 服务端部署完成，即可无缝体验。

### 服务端

Chrome有个 [插件](https://chrome.google.com/webstore/detail/mpbpobfflnpcgagjijhmgnchggcjblin)，可以简单的监测站点是否采用了 HTTP/2， 经过一些主流网站的观察，发现国外站点的部署效率的确要高很多。

Google、Twitter、YouTube目前都采用 HTTP/2 （蓝色⚡️代表H2）

![](https://raw.githubusercontent.com/hectorguo/blog-imgs/master/img/20190608235723.png)

![](https://raw.githubusercontent.com/hectorguo/blog-imgs/master/img/20190608235731.png)

![](https://raw.githubusercontent.com/hectorguo/blog-imgs/master/img/20190608235740.png)

而国内看了下，百度，必应都木有采用，而淘宝采用了SPDY 3.1（可以看作H2的前一代，绿色⚡️）

![](https://raw.githubusercontent.com/hectorguo/blog-imgs/master/img/20190608235749.png)

![](https://raw.githubusercontent.com/hectorguo/blog-imgs/master/img/20190608235757.png)

![](https://raw.githubusercontent.com/hectorguo/blog-imgs/master/img/20190608235805.png)

## 参考

1. [https://http2.github.io/faq/](https://http2.github.io/faq/)
2. [https://http2-push.appspot.com/](https://http2-push.appspot.com/)
3. [http://blog.httpwatch.com/2015/01/16/a-simple-performance-comparison-of-https-spdy-and-http2/](http://blog.httpwatch.com/2015/01/16/a-simple-performance-comparison-of-https-spdy-and-http2/)
4. [https://github.com/molnarg/node-http2](https://github.com/molnarg/node-http2)
5. [https://bjartes.wordpress.com/2015/02/19/creating-a-http2-server-with-node-js/](https://bjartes.wordpress.com/2015/02/19/creating-a-http2-server-with-node-js/)
6. [https://www.futurehosting.com/blog/what-does-http2-mean-for-web-designers/](https://www.futurehosting.com/blog/what-does-http2-mean-for-web-designers/)
7. [https://github.com/GoogleChrome/simplehttp2server](https://github.com/GoogleChrome/simplehttp2server)
8. [https://www.youtube.com/watch?v=r5oT_2ndjms](https://www.youtube.com/watch?v=r5oT_2ndjms)
