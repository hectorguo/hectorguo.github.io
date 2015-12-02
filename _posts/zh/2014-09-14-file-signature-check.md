---
layout: post
title: "利用html5识别文件类型"
published: true
category: zh
tags: ['HTML5','前端开发']
cover: "http://ww4.sinaimg.cn/large/6d0af205jw1evtw72ji9pj20hs0aoq4b.jpg"
---

通常，在文件上传时，我们只会根据文件的扩展名来识别并限制所上传的文件类型。 
比如只允许上传 Excel，那么我们将会查看文件的扩展名是不是`.xls`/`.xlsx`。

但我们知道，在windows系统中，我们可以任意更改文件的扩展名。 那么如果某些用户，将一个zip的压缩文件，改为 `.xls`，在上传时，单单通过扩展名来识别文件，那么它一定是可以通过的。
很多时候，用户都会利用这个漏洞，上传一些看似合法的文件，就如上述场景，将盗版的视频文件改为`.pdf`，跳过审查（某些论坛就是这么干的）。

**需求场景**  

> 小A: "呀，网盘不能上传视频了，咋办？"
> 
> 大B："什么情况？"
> 
> 小A："我一上传mp4的文件，它就提示文件名不合法..."
> 
> 大B："把扩展名改成pdf试下，下载看的时候，再改过来。"
> 
> ...
> 
> 小A："真的可以了诶！ 膜拜大神！"

那么我们是否可以通过某些方法，来识别出这类“非法”文件呢？

#### 如果文件扩展名改了，原本的文件内容会变吗？

当然不会...

我们可以做个实验，比如原本就是一个视频文件，我们改掉扩展名后，再用播放器打开。 测试会发现，播放器一样可以播放该视频。

那么就是说，我们可以通过读取文件内容，来具体识别出文件类型。 

原理在此简单介绍下，其实在每个文件生成时，都会有个[`File Signature` (亦称`magic number`)](http://www.garykessler.net/library/file_sigs.html)。 一般都会在文件头处（即前4 ~ 8位字节）。因此，我们可以将此段读取出来，与File Signature的库对应，即可找到实际的文件类型。

网上很多的解决方法，都是通过将文件上传到服务器之后，通过后端语言读取识别。 

但这样就会产生一定的延迟，如果遇到较大的文件，让用户等待的时间则过长，体验并不好。

在此，着重介绍一下，如果利用HTML5的新特性`FileReader`，直接在浏览器中读取文件内容并识别类型。

### 利用HTML5直接识别文件类型

不废话，上源码

{% highlight html %}
<input type="file" onchange="checkFileType(this.files[0])"></input>

<script>
  function checkFileType(file){
    var slice = file.slice(0,4);      // 读取文件的前4字节，slice方法可以将文件切片
    var reader = new FileReader();    // 创建FileReader实例，注：这个是异步的，可以想象成ajax，只不过读的是文件系统，而不是远程服务器
    reader.readAsArrayBuffer(slice);  // 读取切片文件，此方法读取成功将返回blob格式
    reader.onload = function(e) {
        var buffer = reader.result;          // The result ArrayBuffer
        var view = new DataView(buffer);      // Get access to the result bytes
        var signature = view.getUint32(0, false).toString(16);  // Read 4 bytes, big-endian，return hex string
        switch(signature) {                      // 每个不同的文件类型，都会对应一个唯一的16进制字节串
          case "89504e47": file.verified_type = "image/png"; break;
          case "47494638": file.verified_type = "image/gif"; break;
          case "25504446": file.verified_type = "application/pdf"; break;
          case "504b0304": file.verified_type = "application/zip"; break;
        }
        console.log(file.name, file.verified_type);
  }
</script>
{% endhighlight %}

### 实际应用

好了，上述核心功能已介绍完成。但使用时，会发现，不过的文件类型，它的Signature的要求是不同的。 
有些是前8位字节，有些则是前4位字节。而有些则不是从文件头开始的，需要从512字节开始读取。
更变态的是，同一扩展名会有多个Signature。

因此，在实际运用时，我们并不能只截取文件的前几位字节来识别。那如何可以准备识别文件类型呢？
目前思路如下，单独建立一个库，先根据所上传文件的扩展名，找到它所需要截取的文件部分，读取后对应其Signature，如果不匹配，则说明该文件类型与扩展名是不符合的。

目前，我建立的库如下，分别包含每个文件扩展名对应的Signature，文件截取位置（offset），及截取大小（sizet）。

{% highlight javascript %}
var signature = {
    doc: {
        signature: "D0CF11E0A1B11AE1",
        offset: 0,
        sizet: 8
    },
    docx: {
        signature: "504B030414000600",
        offset: 0,
        sizet: 8
    },
    jpe: {
        signature: "FFD8FFE0",
        offset: 0,
        sizet: 4
    },
    jpeg: {
        signature: "FFD8FFE0",
        offset: 0,
        sizet: 4
    },
    jpg: {
        signature: ["FFD8FFE0","FFD8FFE1","FFD8FFE8"],
        offset: 0,
        sizet: 4
    },
    ...
}
    

{% endhighlight %}

实现如下：

{% highlight javascript %}
  // 通过文件名获取文件的扩展名
  function getFileExtension(fileName) {
      var matches = fileName && fileName.match(/\.([^.]+)$/);
      if (matches) {
        return matches[1].toLowerCase();
      }
      return '';
  }

  // 检测文件类型是否与扩展名相符，并异步返回
  function checkFileSignature(file, onLoad) {
      var ext = getFileExtension(file.name) || 'none',
          fileSign = signatures[ext] || {
              offset: 0,
              sizet: 4
          },
          slice = file.slice(fileSign.offset, fileSign.offset + fileSign.sizet), // slice file from offset to sizet
          reader = new FileReader();
      reader.onload = function(e) {
          var buffer = reader.result, // The result ArrayBuffer
              view = new DataView(buffer), // Get access to the result bytes
              signature, // Read 4 or 8 bytes, big-endian
              isMatch = false; // whether file signature match with the source file type
          
          // get Hex String of file Signatrue, 32bit only contain 4 bytes
          if(view.byteLength == 8){
               signature = view.getUint32(0, false).toString(16) + view.getUint32(4, false).toString(16);
          } else {
              signature = view.getUint32(0, false).toString(16);
          }
          signature = signature.toUpperCase();

          // check signature in file signatures
          if (!jQuery.isArray(fileSign.signature)) {
              fileSign.signature = [fileSign.signature];
          }
          if (jQuery.inArray(signature, fileSign.signature) !== -1) {
              isMatch = true;
          }
          onLoad(isMatch, signature); // async load callbacks
      };
      reader.readAsArrayBuffer(slice); // Read the slice of the file
  }
{% endhighlight %}

### 总结

目前通过纯前端读取文件内容来识别文件类型，可以算是最快最简洁的方法。

但同样也存在一些缺陷：

**1. 部分文件类型的Signature完全一样，并非能完全区分开**

如微软的Office系列文件，`.xlsx`,`.docx`,`.pptx`，其Signature是完全一样的。

拿了几个文件来测试，结果如下：

| 文件      |    测试结果 |
| :-------- | --------:|
| mp4 改 pdf   | 已拦截 |
| zip 改 docx  | 已拦截 |
| zip 改 jpg   | 已拦截 |
| docx 改 zip  | 未拦截 |
| docx 改 xlsx | 未拦截 |

**2. 兼容性问题**

目前支持`HTML5`的`FileReader`方法的浏览器如下：

| Feature   | Firefox (Gecko) | Chrome | Internet Explorer* | Opera* | Safari |
| :-------- | :-------------- | :----- | :----------------- | :----- | :----- |
| Basic support | 3.6 (1.9.2) | 7 | 10 | Not supported | Not supported |

可以看出，IE9及以下版本的浏览器并无法很好的支持。

**3. File Signature的数据**

关于file Signature的数据对应库，可以在[File Signature Database](www.filesignatures.net)查找。目前并不是所有文件都会有对应，只是会一直更新。

### 参考

[1] [FileReader API](https://developer.mozilla.org/zh-CN/docs/Web/API/FileReader)

[2] [Reading the first four bytes of a file](https://www.inkling.com/read/javascript-definitive-guide-david-flanagan-6th/chapter-22/reading-the-first-four-bytes-of)

[3] [FILE SIGNATURES TABLE](http://www.garykessler.net/library/file_sigs.html)

[4] [File Signature Database](http://www.filesignatures.net)
