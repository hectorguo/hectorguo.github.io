---
layout: post
title: 让OS X词典具备保存单词功能 (2017.03.15新增直接导入Evernote)
categories: ['zh']
tags: ['Mac开发','Shell']
published: True
cover: "https://raw.githubusercontent.com/hectorguo/blog-imgs/master/img/20190608235259.png"
---
发现OS X里的词典真的异常方便啊，在MacBook上只需三指轻按触摸板，就能查到页面上的单词释义，而且任何界面都可以查，着实省下了好多查单词时间。可是该词典并不支持添加生词本或者保存历史记录，导致有些生词查了好几次还是没有记住，要是能直接保存并支持导出，那真的就太完美了。搜了一下，真的木有插件类的东西啊，那就自己动手啦。

OS X还有一个自带的工作流制作器Automator，真的是人性化的工具啊。你可以用这个把一系列系统相关的操作合并到一起，生成一个工作流，用的时候一键就搞定。不废话，下面介绍下如何使用Automator制作一个简单的生词本。

### 1. 新建服务
![](https://raw.githubusercontent.com/hectorguo/blog-imgs/master/img/20190608235310.png)

### 2. 选择获得词语定义
在这里，你可以选择你想获得的词典翻译
![](https://raw.githubusercontent.com/hectorguo/blog-imgs/master/img/20190608235319.png)

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

![](https://raw.githubusercontent.com/hectorguo/blog-imgs/master/img/20190608235329.png)

就此服务就完成啦，保存即可。你可以创建两个服务，一个保存英文释义，一个保存中文释义。

![](https://raw.githubusercontent.com/hectorguo/blog-imgs/master/img/20190608235339.png)


### 4. 绑定快捷键
上述步骤完成后，你就可以直接在服务中看到你自定义的工作流。
如果你想方便快捷的保存单词，则直接在`系统偏好设置>键盘>快捷键`中找到对应的服务，自定义快捷键即可。

![](https://raw.githubusercontent.com/hectorguo/blog-imgs/master/img/20190608235349.png)

看下最终成果吧。

![](https://raw.githubusercontent.com/hectorguo/blog-imgs/master/img/20190608235359.png)

如果你想好看一点，当然可以利用正则表达式，把文本格式化一下，接着保存到Evernote中，就可以随身携带背诵啦。

![](https://raw.githubusercontent.com/hectorguo/blog-imgs/master/img/20190608235409.png)

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

![](https://raw.githubusercontent.com/hectorguo/blog-imgs/master/img/20190608235423.png)

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

## 2017.03.15 更新 - 直接导入Evernote：
此工作流可以帮助直接在Evernote中新建一个笔记，并把单词释义导进去，之后可以利用 ‘笔记合并’功能，将多个单词合并到一个笔记中。

合并前：
![before merge](https://raw.githubusercontent.com/hectorguo/blog-imgs/master/img/20190608235437.png)

合并后：
![after merge](https://raw.githubusercontent.com/hectorguo/blog-imgs/master/img/20190608235445.png)

工作流步骤如下：

1. 先设置一个变量叫wordName（用来保存选中的单词）
2. 获取释义
3. 获取变量wordName
4. 执行AppleScript（把如下代码拷贝过去，就可以了）：

{% highlight bash %}
on run {input}
	tell application "Evernote" to activate
	set selectedText to first item of input
	set wordName to second item of input
	tell application "Evernote"
		if (not (notebook named "wordlist" exists)) then
			make notebook with properties {name:"wordlist"}
		end if
		try
			create note title wordName with text selectedText notebook "wordlist"
			synchronize
		end try
	end tell
end run
{% endhighlight %}

在此提供写好的工作流，可以直接下载导入，[下载链接](https://pan.baidu.com/s/1eRJhOtW)