## 目录结构

```
云社区
|-- layout    // 布局
|   `-- index.html       // 用于社区整体布局
|   `-- style.css       
|-- post    // 内容发布区
|      |-- tutorials
|      |-- community
|      |   `-- pagetmpl
|      |       |-- cs
|      |       |-- fin
|      |       |-- form
|      |       |-- home
|      |       |   `-- test.html   // 识别到文件，自动生成栏目Tree
|      |       |-- hr
|      |       |   `-- test.html
|      |       |-- ipd
|      |       |-- it
|      |       |-- list
|      |       |-- sc
|      |       |-- sd
```

## 整体思路
#### 1. 独立模版文件，生成布局
单独模版文件，定义社区布局.  
```html
<!-- layout- index.html -->
<!DOCTYPE html>
<html xmlns="http://www.w3.org/1999/xhtml" xml:lang="en" lang="en-us">
  <body>
      <div class="nav">
            {{ navhead }}
      </div>
      
    {{ sidebar }}

    <div class="content container">
      {{ content }}
    </div>

  </body>
</html>

```

#### 2. 根据内容页面，自动生成栏目树
每个页面一个实体文件(wml)，根据目录结构，自动生成左侧栏目Tree。  
首页Metro区块，亦可根据内容生成。



#### 3. 内容发布器集合HTML、WML，支持DEMO+文本混编
撰写API的同时，支持wml混编，方便同时编辑富文本+DEMO。
```xml
<h3>国际化</h3>

<p>用于多语言页面的维护</p>

<h4>Demo如下</h4>

<?xml version="1.0" encoding="UTF-8"?>
<page name="i18n" version="1.0">
	<!-- page inits -->
	<pageinit>
		<action type="set">
			<extend name="input">{$obj : "$URL"}</extend>
			<extend name="output">$VO.i18nListModel</extend>
		</action>	
		<!--<action type="service" method="saasI18nGetInfoWithPage">
			<extend name="input">{$obj : "$VO.i18nModel"}</extend>
			<extend name="output">$DM.i18nListModel</extend>
		</action>-->
	</pageinit>
	<!-- page models -->
    ...
```

Markdown形式：

```markdown
### 国际化

用于多语言页面的维护

Demo如下

<?xml version="1.0" encoding="UTF-8"?>
<page name="i18n" version="1.0">
	<!-- page inits -->
	<pageinit>
		<action type="set">
			<extend name="input">{$obj : "$URL"}</extend>
			<extend name="output">$VO.i18nListModel</extend>
		</action>	
		<!--<action type="service" method="saasI18nGetInfoWithPage">
			<extend name="input">{$obj : "$VO.i18nModel"}</extend>
			<extend name="output">$DM.i18nListModel</extend>
		</action>-->
	</pageinit>
	<!-- page models -->
    ...
```

#### 4. 试一试内嵌式集成

<p data-height="268" data-theme-id="0" data-slug-hash="IdGKH" data-default-tab="result" data-user="sevilayha" class='codepen'>See the Pen <a href='http://codepen.io/sevilayha/pen/IdGKH/'>Google Material Design Input Boxes</a> by Chris Sevilleja (<a href='http://codepen.io/sevilayha'>@sevilayha</a>) on <a href='http://codepen.io'>CodePen</a>.</p>
<script async src="//codepen.io/assets/embed/ei.js"></script>

```html
<p data-height="268" data-theme-id="0" data-slug-hash="IdGKH" data-default-tab="result" data-user="sevilayha" class='codepen'>See the Pen <a href='http://codepen.io/sevilayha/pen/IdGKH/'>Google Material Design Input Boxes</a> by Chris Sevilleja (<a href='http://codepen.io/sevilayha'>@sevilayha</a>) on <a href='http://codepen.io'>CodePen</a>.</p>
<script async src="//codepen.io/assets/embed/ei.js"></script>
```

