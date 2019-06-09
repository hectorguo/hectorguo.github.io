---
layout: post
title: 列表响应式布局--三种CSS方案
categories: ['zh']
tags: ['CSS','前端开发']
published: True
cover: "https://raw.githubusercontent.com/hectorguo/blog-imgs/master/img/20190608214019.png"
---

最近需要做一些带图片的布局，适用于桌面及移动端。参考了一些网站的实践方案，在此做个总结。

整体思路: 
1. 每张图片高宽一致，居中对齐
2. 根据设备宽度不同，自动换行

总结下来，便有了三种方案。
1. [流式布局—— Float](#float)
2. [内联布局—— Inline-block](#inline-block)
3. [CSS3新特性—— Flexbox](#css3-flexbox)

简单写下三种布局的实现：

### 1.流式布局—— Float

流式布局是最为常见的一种方案，但很多时候也不太愿意使用，因为浮动的样式会造成内部内容高宽的不可控，在此参考了[dribble](dribbble.com)的方案，利用`:after`额外添加内容，并清除浮动，保证高度的可控。

{% highlight css %}
/*float layout*/
.float {
  max-width: 1200px;
  margin: 0 auto;
}
.float:after {
  content: ".";
  display: block;
  height: 0;
  clear: both;
  visibility: hidden;
}
.float-item {
  float: left;
}
{% endhighlight %}

### 2.内联布局—— Inline-block

内联方式会有一个缺点，就是每个item间会默认留一个空格的间距，即最终的间距＝外边距＋1个空格。

{% highlight css %}
/*inline-block*/
.inline-b {
  max-width:1200px;
  margin:0 auto;
}
.inline-b-item {
  display: inline-block;
}
{% endhighlight %}

### 3.CSS3新特性—— Flexbox

Flexbox为CSS3标准的新特性，它的加入也让页面的布局更丰富和灵活。其功能较多，在此不做详细介绍，详情可查看 [CSS-Tricks](https://css-tricks.com/snippets/css/a-guide-to-flexbox/) 讲解。
{% highlight css %}
/*Flexbox*/
.flex {
  padding: 0;
  margin: 0;
  list-style: none;
  
  display: -webkit-box;
  display: -moz-box;
  display: -ms-flexbox;
  display: -webkit-flex;
  display: flex;
  
  -webkit-flex-flow: row wrap;
  justify-content: space-around;
}
{% endhighlight %}

最终效果见下方：

<p data-height="653" data-theme-id="20354" data-slug-hash="BoZEyW" data-default-tab="result" data-user="hectorguo" class='codepen'>See the Pen <a href='http://codepen.io/hectorguo/pen/BoZEyW/'>Responsive List Layout (Flexbox vs Float vs Inline-Block)</a> by Hector Guo (<a href='http://codepen.io/hectorguo'>@hectorguo</a>) on <a href='http://codepen.io'>CodePen</a>.</p>
<script async src="//assets.codepen.io/assets/embed/ei.js"></script>


最后列个表，方便对比。

|  | 适用场景 | 兼容性 | 参考网站 |
| ------------- | ------------- | ------------- | ------------- |
| Float | 常规实现，适用于列表item较多，强调整体对齐 | 适用所有 | [dribble](dribbble.com) |
| Inline-block | 对各个item间宽度精度要求不高，item较少 | IE8+ |  |
| Flexbox | 对移动端有针对性设计，强调居中对齐 | IE10+（IE10需加前缀-ms-）| [CSS-tricks](https://css-tricks.com/snippets/css/a-guide-to-flexbox/) |

