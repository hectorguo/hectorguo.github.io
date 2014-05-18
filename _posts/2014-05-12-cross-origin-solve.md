---
layout: post
title: 前端跨域问题解决方法集锦
published: true
---

### 何为跨域（CORS）?
> 所謂跨站HTTP請求(Cross-site HTTP request)是指發出請求所在網域不同於請求所指向之網域的HTTP請求，例如網域A (http://domaina.example) 的網頁載入一個<img>元素向網域B(http://domainb.foo) 請求影像資源(http://domainb.foo/image.jpg) 。這種作法在現今各網頁相當常見，網頁常常會載入其他網站資源，像是CSS樣式表、影像、程式碼等等資源。

即基于安全性考虑，在javascript中制定了[同源策略（Same Origin Policy）](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Same_origin_policy_for_JavaScript)。简单来讲，只有拥有相同的协议（同为http或https） + 端口（port）+ 主机(domain) 才属于同一个源（origin）,只有此类才可以进行资源的交互。

------------------

### 外网跨域
针对外网的跨域，即在本地(localhost) / 你的站点 中，想获取外部的数据，一般为其他的开放API。如[google map api](https://developers.google.com/maps/?hl=zh-cn)，但此时的数据与 你的站点不属于同源，当进行请求时，浏览器则会提示**across origin error**。
随着html5的诞生，W3C也开始考虑对跨域的请求方式进行规范。
#### 解决方法：
1. **XMLHttpRequest Level 2**（简称**XHR**） —— 新版本的http(s)通信方式
    由于老版本XHR收到`同源策略`限制，只能向同一域名的服务器请求数据。
    因此新版本，针对老版本的缺点，做了改进，支持 `跨域请求`。
    但前提是浏览器必须支持这个功能，而且服务器端必须同意这种"跨域"（[Access-Control-Allow-Origin设置](https://dvcs.w3.org/hg/cors/raw-file/tip/Overview.html)）。如果能够满足上面的条件，则代码的写法与不跨域的请求完全一样。
    
    ``` javascript
        var xhr = new XMLHttpRequest();
        xhr.open('GET', 'http://maps.googleapis.com/maps/api/distancematrix/json');
        xhr.onload = function(e) {
          var data = JSON.parse(this.response);
          ...
        }
        xhr.send();
    ```
    新版XHR中，新增了`xhr.responseType`,`xhr.response`属性，可以告诉浏览器，请求后返回什么格式的数据，使我们可以更方便的抓取二进制blob形式的文件。详情可参考[XMLHttpRequest2 新技巧](http://www.html5rocks.com/zh/tutorials/file/xhr2/)。

2. **JSONP** —— 新型的简单的解决方法
    **JSONP**可以克服 **XMLHttpRequest** 的同源限制，简单的获取到不同源的数据。
    原理很简单，就是script标签 + callback 。（因为 script 并不受同源限制的影响）
    
    ``` javascript
        script = document.createElement("script");
        script.type = "text/javascript";
        script.src = "http://www.someWebApiServer.com/some-data";
    ```
    执行过后，将会得到下面的数据：
    
            <script>{['some string 1', 'some data', 'whatever data']}</script>
    
    但是这种形式，有些不方便抓取到里面的数组，因此改进如下：
    
    ``` javascript
        script = document.createElement("script");
        script.type = "text/javascript";
        script.src = "http://www.someWebApiServer.com/some-data?callback=my_callback";
    ```
    注意到了吗，我们添加了一个 my_callback 函数。因此，当服务器接受到你的请求时，会找到该callback参数，并相应生成下面的形式：
    
    ``` javascript
        my_callback({['some string 1', 'some data', 'whatever data']});
    ```
    这样的有利之处在于，我们所获取的数据会自动回调到 my_callback中。
    
    **原生javascript用法**：
    
    ``` html
    <html>
        <head>
        </head>
        <body>
            <div id = 'twitterFeed'></div>
            <script>
            function myCallback(dataWeGotViaJsonp){
                var text = '';
                var len = dataWeGotViaJsonp.length;
                for(var i=0;i<len;i++){
                    twitterEntry = dataWeGotViaJsonp[i];
                    text += '<p><img src = "' + twitterEntry.user.profile_image_url_https +'"/>' + twitterEntry['text'] + '</p>'
                }
                document.getElementById('twitterFeed').innerHTML = text;
            }
            </script>
            <script type="text/javascript" src="http://twitter.com/status/user_timeline/padraicb.json?count=10&callback=myCallback"></script>
        </body></html>
    
    ```
    
    **jquery用法**：
    
    ``` html
    <html>
        <head>
            <script type="text/javascript" src="https://ajax.googleapis.com/ajax/libs/jquery/1.6.2/jquery.min.js"></script>
            <script>
                $(document).ready(function(){
                    $.ajax({
                        url: 'http://twitter.com/status/user_timeline/padraicb.json?count=10',
                        dataType: 'jsonp',
                        success: function(dataWeGotViaJsonp){
                            var text = '';
                            var len = dataWeGotViaJsonp.length;
                            for(var i=0;i<len;i++){
                                twitterEntry = dataWeGotViaJsonp[i];
                                text += '<p><img src = "' + twitterEntry.user.profile_image_url_https +'"/>' + twitterEntry['text'] + '</p>'
                            }
                            $('#twitterFeed').html(text);
                        }
                    });
                })
            </script>
        </head>
        <body>
            <div id = 'twitterFeed'></div>
        </body></html>
            
    ```

3. **利用Chrome 插件解决跨域** —— 无需借助服务器声明，无障碍跨域
    如果我不想这么麻烦，我只想批量抓取外网的一些数据结果，是否有简单的方法呢？
    当然，目前Chrome扩展中为了方便插件获取不同域的数据，提供了解决方法。
    即只需在`manifest.json`文件中的**"permission"**中声明，再用xmlhttpRequest即可。详情可参考[Chrome扩展开发手册](http://open.chrome.360.cn/html/dev_xhr.html)


        {
          "name": "My extension",
          ...
          "permissions": [
            "http://www.google.com/"
          ],
          ...
        }

-----------------

### 本地跨域
有时，我们在本地运行页面时，也会遇到across origin error的情况。
![Alt text](./2014-05-12_204749.jpg)

这个是因为浏览器的限制。 目前chrome、firfox、opera都出现了这种情况，因为有安全沙箱，它们认为加载本地其它文件为跨域访问（但是使用IE8就不会出现这种错误）。
为了解决该问题，解决方法有两种：
1. **设置Chrome启动参数**
    在启动Chrome时，添加一个参数：**--allow-file-access-from-files**
    ![Alt text](./2014-05-12_205659.jpg)

    每次启动Chrome都使用该快捷方式，即可解决本地跨域问题
2. **启动Apache设置虚拟主机**
    Apache中含有虚拟主机的功能，即可以将不同路径下的资源(file://d:/projectA ; file://c:/projectB)，设置成同源(localhost/projectA; localhost/projectB),这样引用时，由于都处在 localhost下，因此不会出现跨域问题。

        
        #设置虚拟主机与虚拟目录  
        #配置文件 Apache安装目录 \conf\extra\httpd-vhosts.conf
        <VirtualHost *:80>
            UseCanonicalName Off
            ServerName localhost
            DocumentRoot "D:/projectA"
            <Directory "D:/projectB">
        	    Options Indexes FollowSymLinks
        	    AllowOverride None
        	    Order allow,deny
        	    Allow from all
        	</Directory>
        </VirtualHost>
        
        Alias /projectA "D:/projectA"   
        Alias /projectB "C:/projectB"

详细配置可参考[Apache VirtualHost配置](http://httpd.apache.org/docs/2.2/zh-cn/vhosts/)


####参考
[1] [XMLHttpRequest2 新技巧](http://www.html5rocks.com/zh/tutorials/file/xhr2/)
[2] [Chrome扩展开发手册](http://open.chrome.360.cn/html/dev_xhr.html)
[3] [解决Chrome跨域的方法](http://hi.baidu.com/qf_soft/item/01a3bcda48ee7cca1a72b486)
[4] [JavaScript的同源策略](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Same_origin_policy_for_JavaScript)
[5] [XMLHttpRequest Level 2 使用指南](http://www.ruanyifeng.com/blog/2012/09/xmlhttprequest_level_2.html)
