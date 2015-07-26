---
layout: post
title: "Jquery Tips(1)"
published: true
category: en
tags: javascript
---

本篇主要讲解jquery中一些方法的性能及注意事项，全是项目中遇到的问题总结，怕以后会忘记，遂在此记录一下。
### 1. `$( )`相对选择器 与  `find`
```javascript
$('#id', '.container')  
$('.container').find('#id')
// 两者比较，前者会 先解析转换成 后者， 因此 find 的效率是最高的。 
```

### 2. jquery 获取整个html，即outerHTML

```javascript
$('<div>').append($('#item-of-interest').clone()).html(); 
```

jquery 1.6后，可以用这个

```javascript
$('#element-of-interest').prop('outerHTML');
```

如果要替换当前的dom，则直接 `$('#element-of-interest').replaceWith('<div>xxx</div>')` 即可。 该方法会覆盖整个dom（outerHTML）。


### 3. 清空某dom下的html
`$('xxx').empty() `
此方法 比 `$('xxx').html('')` 更高效

### 4. 遍历对象 `$.each()`  vs  `for ... in`
测试发现，遍历对象，`$.each()` 比 `for ... in` 更快  
[jsperf测试结果](http://jsperf.com/foreach-vs-jquery-each/9)  

```javascript
$.each(simplePager, function(index, val) {
    html.push('<div class="pager-group">');
    simplePager[index](html, op);  //可用此遍历对象，执行对象内的函数
    html.push('</div>');
});
```
### 5. jquery.val() 与 onchange
`jquery.val()` 方法不会触发 Select ，input，textarea 等标签的onchange事件。
