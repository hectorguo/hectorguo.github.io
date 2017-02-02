---
layout: post
title: 让OS X词典具备保存单词功能
categories: ['zh']
tags: ['Mac开发','Shell']
published: True
cover: "https://ww2.sinaimg.cn/large/6d0af205jw1exeycry7e0j20nw0ay42t.jpg"
---
发现OS X里的词典真的异常方便啊，在MacBook上只需三指轻按触摸板，就能查到页面上的单词释义，而且任何界面都可以查，着实省下了好多查单词时间。可是该词典并不支持添加生词本或者保存历史记录，导致有些生词查了好几次还是没有记住，要是能直接保存并支持导出，那真的就太完美了。搜了一下，真的木有插件类的东西啊，那就自己动手啦。

OS X还有一个自带的工作流制作器Automator，真的是人性化的工具啊。你可以用这个把一系列系统相关的操作合并到一起，生成一个工作流，用的时候一键就搞定。不废话，下面介绍下如何使用Automator制作一个简单的生词本。

### 1. 新建服务
![](https://ww1.sinaimg.cn/large/6d0af205jw1exey4g3zp0j20fo0evdi7.jpg)

### 2. 选择获得词语定义
在这里，你可以选择你想获得的词典翻译
![](https://ww2.sinaimg.cn/large/6d0af205jw1exey4z2zruj20r907a40p.jpg)

### 3. 创建Shell脚本
因为需要有个文档来保存获取的单词定义，因此需要用shell来操作。在左侧搜索`shell`即可看到`运行Shell脚本`的工作流，拖拽过来，输入下面代码即可：

{% highlight bash %}
  # 将单词释义保存到桌面的wordlist.txt文件下，若文件存在，则直接追加
  FILE=$HOME/Desktop/wordlist.txt
  if [ ! -f $FILE ]; then
  touch $FILE
  fi
  echo -e "\n$1" >> $FILE
{% endhighlight %}

![](https://ww4.sinaimg.cn/large/6d0af205jw1exey5a1o61j20h80as407.jpg)

就此服务就完成啦，保存即可。你可以创建两个服务，一个保存英文释义，一个保存中文释义。

![](https://ww3.sinaimg.cn/large/6d0af205jw1exey6xcyggj209608qmxy.jpg)


### 4. 绑定快捷键
上述步骤完成后，你就可以直接在服务中看到你自定义的工作流。
如果你想方便快捷的保存单词，则直接在`系统偏好设置>键盘>快捷键`中找到对应的服务，自定义快捷键即可。

![](https://ww4.sinaimg.cn/large/6d0af205jw1exey5r4phzj20hs0afq5d.jpg)

看下最终成果吧。

![](https://ww2.sinaimg.cn/large/6d0af205jw1exey7a3f5bj20h50a3q8o.jpg)

如果你想好看一点，当然可以利用正则表达式，把文本格式化一下，接着保存到Evernote中，就可以随身携带背诵啦。

![](https://ww2.sinaimg.cn/large/6d0af205jw1exey7qp096j20io0j1dm8.jpg)

最后附上做好的工作流，方便各位导入。
[网盘下载](http://pan.baidu.com/s/1bn7a8n9) 

## 2017.01.14 更新：
很多人来问如何用正则表达式来格式化数据。在此分享下新的Shell脚本，可以直接输出格式化后的`.html`文件，用浏览器打开即可看。

将第3步的Shell脚本替换为如下即可。

{% highlight bash %}
  # 将单词释义保存到桌面的wordlist.html文件下，若文件不存在，则初始化文件（填充html的标签）
  FILE=$HOME/Desktop/wordlist.html
  INIT_STRING='<!DOCTYPE html><html lang="en"><head><meta charset="UTF-8"><title>MAC-Wordlist</title></head><body><table></table></body></html>'
  # 格式化单词释义（把单词，音标，解释分离开）
  RESULT=$(echo -e $1 | sed -E 's#([a-zA-Z]+) (\|[^\|]+\|)(.*)#<tr><td>\1</td><td>\2</td><td>\3</td></tr>#g')
  if [ ! -f $FILE ]; then
  echo $INIT_STRING > $FILE
  fi
  sed -Ei '' "s#(</table>)#$RESULT\1#g" $FILE
{% endhighlight %}

最终截图如下：

![](https://ww3.sinaimg.cn/large/6d0af205gw1fbqs0hz1p3j20wd0ca135.jpg)

顺便分享下Shell脚本的小坑，使用`sed`来用正则表达式替换字符串时，如果替换的字符串里还有`/`等特殊字符（与正则表达式冲突）导致无法替换时，可将命令行里的`/`用`#`或`@`等其他特殊字符代替即可。

如

{% highlight bash %}
  RESULT="<td>test</td>"
  sed -Ei '' "s/(<\/table>)/$RESULT\1/g" $FILE
{% endhighlight %}

上述会报错，因为`RESULT`变量里的`</td>`中的`/`与`sed`里用的`/`冲突了。
可改为：

{% highlight bash %}
  RESULT="<td>test</td>"
  sed -Ei '' "s#(</table>)#$RESULT\1#g" $FILE
{% endhighlight %}