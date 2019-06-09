---
layout: post
title: 高清图标方案对比:SVG与字体图标
categories: ['zh']
tags: ['CSS','前端开发']
published: True

---
最近读了 [SitePoint](http://www.sitepoint.com/icon-fonts-vs-svg-debate/) 的文章，发现`SVG`的图标方案有很多地方优于字体图标方案，如语义性良好，以及无损还原显示效果。
于是兴冲冲的准备把自己的站点的图标都改成SVG方案，可折腾了许久后，发现会出现各种各样的问题，并不是想象中如此完美，在此做个对比总结，分享给有需要的人。

* TOC
{:toc}

## 简评

目前所有图标方案，每一个都会存在这样或那样的缺陷，我们需要的是针对不同场景，不同需求，来选用不同的方案。

|  | 优点 | 缺点 |
| ------------- | ------------- | --- |
| Icon Fonts | 向下兼容所有浏览器(IE6+)；引用方便，对单个图标的控制也方便 | 语义性欠缺；不同浏览器渲染方式不同，存在锯齿现象 |
| SVG | 无损还原，显示清晰；语义性良好 | 原生只支持到IE9+；方案众多，不同方案都存在一定缺陷 |

## 字体图标方案

字体图标方案提出的本意是为了解决切图带来的不便，以及对高清设备显示模糊现象的问题。 

![](https://raw.githubusercontent.com/hectorguo/blog-imgs/master/img/20190608235502.png)

因此引用了矢量图标，并采用类似`<icon class="icon-user"></icon>`的方式方便引用。
但后来发现该方案在一些复杂图标的显示上并不是那么好，原因是因为不同浏览器对字体的渲染方式不同，抗锯齿的效果欠佳，详细的可见[这篇文章的说明](http://isux.tencent.com/svg-icon-part-one.html)。
并且由于缺乏语义性，对搜索引擎或是其他阅读设备的支持并不友好。如下面这个例子：

> 请点击 <span class="iconfont">&#xe91f;</span> 来分享

如果使用某些可口述的阅读设备来读这段文字，那么就会变成如下结果：

> 请点击来分享

这类问题就如同`<img>`标签不添加`alt`属性一样。
而如果采用svg方案，它会自带有`title`功能，即仍可以把图标用语音描述出来。

<p data-height="132" data-theme-id="20354" data-slug-hash="LpXwmq" data-default-tab="result" data-user="hectorguo" class='codepen'>See the Pen <a href='http://codepen.io/hectorguo/pen/LpXwmq/'>LpXwmq</a> by Hector Guo (<a href='http://codepen.io/hectorguo'>@hectorguo</a>) on <a href='http://codepen.io'>CodePen</a>.</p>
<script async src="//assets.codepen.io/assets/embed/ei.js"></script>

当然，字体图标也可以采用如下方式，解决这类不友好的问题。

```
请点击
<icon class="icon-share">
  <span class="sr-only">"分享"</span>
</icon>
来分享
```

## SVG 6种方案实现

### 1. CSS sprite

{% highlight html %}
<!-- 方案1:CSS Sprite -->
<style>
.sprite-icon {
    display: inline-block;
    background-repeat: no-repeat;
    background-image: url(//hectorguo.com/css/fonts/svg/sprite.svg);
}

.sprite-icon.icon-share2 {
    width: 32px;
    height: 32px;
    background-position: 0 0;
}

.sprite-icon.icon-feed2 {
    width: 32px;
    height: 32px;
    background-position: -64px 0;
}
</style>

<span class="sprite-icon icon-share2"></span>
<span class="sprite-icon icon-feed2"></span>
{% endhighlight %}

### 2. Background Image (路径引用)

{% highlight html %}
<!-- 方案2:Background Image (路径引用) -->
<style>
.path-icon {
    display: inline-block;
}

.path-icon.icon-share2 {
    width: 32px;
    height: 32px;
    background-image: url(//hectorguo.com/css/fonts/svg/SVG/share2.svg)
}

.path-icon.icon-feed2 {
    width: 32px;
    height: 32px;
    background-image: url(//hectorguo.com/css/fonts/svg/SVG/feed2.svg)
}
</style>

<span class="path-icon icon-share2"></span>
<span class="path-icon icon-feed2"></span>
{% endhighlight %}

### 3. Background Image - Data Uri (直接嵌入SVG)

{% highlight html %}
<!-- 方案3:Background Image - Data Uri (直接嵌入SVG) -->
<style>
.embed-icon {
    display: inline-block;
}

.embed-icon.icon-share2 {
    width: 32px;
    height: 32px;
    background-image: url('data:image/svg+xml;utf8,<svg version="1.1" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="32" height="32" viewBox="0 0 32 32"><path fill="#828282" d="M27 22c-1.411 0-2.685 0.586-3.594 1.526l-13.469-6.734c0.041-0.258 0.063-0.522 0.063-0.791s-0.022-0.534-0.063-0.791l13.469-6.734c0.909 0.94 2.183 1.526 3.594 1.526 2.761 0 5-2.239 5-5s-2.239-5-5-5-5 2.239-5 5c0 0.269 0.022 0.534 0.063 0.791l-13.469 6.734c-0.909-0.94-2.183-1.526-3.594-1.526-2.761 0-5 2.239-5 5s2.239 5 5 5c1.411 0 2.685-0.586 3.594-1.526l13.469 6.734c-0.041 0.258-0.063 0.522-0.063 0.791 0 2.761 2.239 5 5 5s5-2.239 5-5c0-2.761-2.239-5-5-5z"></path></svg>')
}

.embed-icon.icon-feed2 {
    width: 32px;
    height: 32px;
    background-image: url('data:image/svg+xml;utf8,<svg version="1.1" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="32" height="32" viewBox="0 0 32 32"><path fill="#828282" d="M4.259 23.467c-2.35 0-4.259 1.917-4.259 4.252 0 2.349 1.909 4.244 4.259 4.244 2.358 0 4.265-1.895 4.265-4.244-0-2.336-1.907-4.252-4.265-4.252zM0.005 10.873v6.133c3.993 0 7.749 1.562 10.577 4.391 2.825 2.822 4.384 6.595 4.384 10.603h6.16c-0-11.651-9.478-21.127-21.121-21.127zM0.012 0v6.136c14.243 0 25.836 11.604 25.836 25.864h6.152c0-17.64-14.352-32-31.988-32z"></path></svg>')
}
</style>

<span class="embed-icon icon-share2"></span>
<span class="embed-icon icon-feed2"></span>
{% endhighlight %}

### 4. SVG `<use>` - 内嵌

{% highlight html %}
<!-- 方案4:SVG <use> - 内嵌 -->
<svg style="display:none" version="1.1" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink">
    <defs>
        <symbol id="icon-share2" viewBox="0 0 1024 1024">
            <title>share2</title>
            <path class="path1" d="M864 704c-45.16 0-85.92 18.738-115.012 48.83l-431.004-215.502c1.314-8.252 2.016-16.706 2.016-25.328s-0.702-17.076-2.016-25.326l431.004-215.502c29.092 30.090 69.852 48.828 115.012 48.828 88.366 0 160-71.634 160-160s-71.634-160-160-160-160 71.634-160 160c0 8.622 0.704 17.076 2.016 25.326l-431.004 215.504c-29.092-30.090-69.852-48.83-115.012-48.83-88.366 0-160 71.636-160 160 0 88.368 71.634 160 160 160 45.16 0 85.92-18.738 115.012-48.828l431.004 215.502c-1.312 8.25-2.016 16.704-2.016 25.326 0 88.368 71.634 160 160 160s160-71.632 160-160c0-88.364-71.634-160-160-160z"></path>
        </symbol>
        <symbol id="icon-feed2" viewBox="0 0 1024 1024">
            <title>feed2</title>
            <path class="path1" d="M136.294 750.93c-75.196 0-136.292 61.334-136.292 136.076 0 75.154 61.1 135.802 136.292 135.802 75.466 0 136.494-60.648 136.494-135.802-0.002-74.742-61.024-136.076-136.494-136.076zM0.156 347.93v196.258c127.784 0 247.958 49.972 338.458 140.512 90.384 90.318 140.282 211.036 140.282 339.3h197.122c-0.002-372.82-303.282-676.070-675.862-676.070zM0.388 0v196.356c455.782 0 826.756 371.334 826.756 827.644h196.856c0-564.47-459.254-1024-1023.612-1024z"></path>
        </symbol>
    </defs>
</svg>

<style>
.use-embed-icon {
    width: 32px;
    height: 32px;
    fill: #828282;
}
</style>

<svg class="use-embed-icon icon-share2">
    <use xlink:href="#icon-share2"></use>
</svg>
<svg class="use-embed-icon icon-feed2">
    <use xlink:href="#icon-feed2"></use>
</svg>
{% endhighlight %}

### 5. SVG `<use>` - 外部引用

{% highlight html %}
<!-- 方案5:SVG <use> - 外部引用 -->
<style>
.use-path-icon {
    width: 32px;
    height: 32px;
    fill: #828282;
}
</style>

<svg class="use-path-icon icon-share2">
    <use xlink:href="//hectorguo.com/css/fonts/svg/symbol-defs.svg#icon-share2"></use>
</svg>
<svg class="use-path-icon icon-feed2">
    <use xlink:href="//hectorguo.com/css/fonts/svg/symbol-defs.svg#icon-feed2"></use>
</svg>
{% endhighlight %}

### 6. `<img>`标签

{% highlight html %}
<!-- 方案6:<img> -->
<img src="//hectorguo.com/css/fonts/svg/SVG/share2.svg" alt="share">
<img src="//hectorguo.com/css/fonts/svg/SVG/feed2.svg" alt="feed">
{% endhighlight %}

### 方案汇总

<p data-height="500" data-theme-id="20354" data-slug-hash="rOQXKd" data-default-tab="result" data-user="hectorguo" class='codepen'>See the Pen <a href='http://codepen.io/hectorguo/pen/rOQXKd/'>rOQXKd</a> by Hector Guo (<a href='http://codepen.io/hectorguo'>@hectorguo</a>) on <a href='http://codepen.io'>CodePen</a>.</p>
<script async src="//assets.codepen.io/assets/embed/ei.js"></script>

## SVG 6种方案优劣对比

6种svg方案都存在些缺陷，对比如下：
<table>
  <thead>
    <tr>
      <th>&nbsp;</th>
      <th>优点</th>
      <th>缺点</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>1.CSS sprite</td>
      <td><ol><li>单一http请求</li><li>可用<code>background-size</code>调整大小（整体调整）</li><li>易于引用（如<code>&lt;span class="icon icon-chrome"&gt;&lt;/span&gt;</code>）</li></ol></td>
      <td>对单个图标调整大小繁琐（需要同时调整<code>background-size</code>和<code>background-position</code>）</td>
    </tr>
    <tr>
      <td>2.Background Image (外部引用)</td>
      <td><ol><li>可用<code>background-size</code>调整大小（个体调整）</li><li>易于引用(同上)</li></ol></td>
      <td>多个http请求</td>
    </tr>
    <tr>
      <td>3.Background Image (直接嵌入SVG)</td>
      <td><ol><li>http请求减少;</li><li>易于引用(同上)</li></ol></td>
      <td><ol><li>性能欠佳（CSS样式表文件变大，若放入html中则不易做缓存）</li><li>  兼容性不佳（IE9+？）</li></ol></td>
    </tr>
    <tr>
      <td>4.SVG <code>&lt;use&gt;</code> - 外部引用</td>
      <td><ol><li>单个图标大小调整灵活</li><li>易于引用(<code>&lt;svg&gt;&lt;use xlink:href="def.svg#icon--feed"/&gt;&lt;/svg&gt;</code>)</li><li>单独样式控制</li><li>易于缓存</li></ol></td>
      <td><ol><li>声明较烦琐</li><li>兼容性问题（IE9、10下外部引用svg文件存在问题）</li></ol></td>
    </tr>
    <tr>
      <td>5.SVG <code>&lt;use&gt;</code> - 内嵌</td>
      <td><ol><li>单个图标大小调整灵活</li><li>易于引用(<code>&lt;svg&gt;&lt;use xlink:href="#icon--feed"/&gt;&lt;/svg&gt;</code>)</li></ol></td>
      <td><ol><li>声明较烦琐</li><li>性能欠佳（每个html文件都存在svg文件，不易缓存）</li></ol></td>
    </tr>
    <tr>
      <td>6.<code>&lt;img&gt;</code></td>
      <td><ol><li>单个图标大小调整灵活</li><li>易于引用(<code>&lt;img src="xxx.svg"&gt;</code>)</li></ol></td>
      <td><ol><li>多个http请求</li><li>样式调整困难</li></ol></td>
    </tr>
  </tbody>
</table>

## 总结

综上所述，没能找到一个最佳的svg方案，之前采用方案4，但发现引用起来并没有字体图标方案来的简单，为了引用一个图标，着实添加了不少代码量，而且由于是在html中引用，如果在不同的路径下统一用相对路径的话（尤其针对Jekyll这类静态博客系统），会带来不少问题，采用绝对路径那引用起来更加麻烦。 且根据[这篇文章](http://blog.pictonic.co/post/32869817328/svgs-are-cool-but-icon-fonts-are-just-10-of)，当随着图标的增多，svg的文件大小显然要高于字体。

![](https://raw.githubusercontent.com/hectorguo/blog-imgs/master/img/20190608235516.png)

因此，后来又改回了字体图标方案。当然，如果你的图标不多，又恰好想新增几个图标上去，显然用svg直接放进html里会更方便，也省去了重新合并字体的麻烦。
如果各位有什么其他更好的方案，欢迎在下面评论探讨。

## 参考
1. [Implementing an SVG Icon System](http://damonbauer.me/implementing-svg/)
2. [高清ICON SVG解决方案](http://isux.tencent.com/svg-icon-part-one.html)
3. [The Great Icon Debate: Fonts Vs SVG](http://www.sitepoint.com/icon-fonts-vs-svg-debate/)
4. [Inline SVG vs Icon Fonts](https://css-tricks.com/icon-fonts-vs-svg/)
5. [svgs are cool, but icon fonts are just 10% of their file size](http://blog.pictonic.co/post/32869817328/svgs-are-cool-but-icon-fonts-are-just-10-of)
