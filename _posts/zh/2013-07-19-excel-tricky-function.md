---
layout: post
title: 工作中Excel里两个实用功能分享——vlookup函数及数据透视表
categories: ['zh']
tags: ['效率提升']
published: True
cover: "http://ww4.sinaimg.cn/large/6d0af205jw1ewrhx7vw23j20gs08m400.jpg"
---

工作一段时间，发现Excel里有两个功能最常用，分享下使用场景及方法。

### Vlookup运用

#### 使用场景：如果两张表中有相同的一列，但顺序已打乱，如何将两张表的值合并？
如下，如何将表2中的“联系方式”，与表一中的根据姓名进行快速匹配合并？
<p>        <span><img src="http://img.blog.csdn.net/20130718235102828" border="0" alt="">&nbsp; &nbsp;<img src="http://img.blog.csdn.net/20130718235147406" border="0" alt=""><br>        </span></p>
 
**解决方法**：
使用vlookup函数。详细方法如下，
![vlookup](http://img.blog.csdn.net/20130718235133859)

注意，匹配区域的第一列必须是关键字才能匹配到。 
![vlookup-2](http://img.blog.csdn.net/20130718235236343) 

匹配成功的结果：
![vlookup-3](http://img.blog.csdn.net/20130718235242484) 

-----------------

### 数据透视表的使用

#### 使用场景：针对庞大的数据，如何进行多维度的统计？
如下图数据，如果想多维度统计，如按**国家**统计**消费总金额**，再看每个国家下**每个省份**的消费总金额，再看这个省份下**具体有哪些人**。
![data](http://img.blog.csdn.net/20130718235245984) 

**a.按国家统计，每个国家下的消费总金额。**
步骤一：
![data2](http://img.blog.csdn.net/20130718235304625) 

步骤二：
![data3](http://img.blog.csdn.net/20130718235310171) 

**b.细分到每个省份下的金额。**
![data4](http://img.blog.csdn.net/20130718235520390) 

**c.显示省份下的人的姓名**
![data5](http://img.blog.csdn.net/20130718235528203) 

结果如下图：
![data6](http://img.blog.csdn.net/20130718235533406) 

**d.显示每个省份下的人数**
![data7](http://img.blog.csdn.net/20130718235538453) 
