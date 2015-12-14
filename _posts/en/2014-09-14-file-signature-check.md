---
layout: post
title: "How to detect File Type through HTML5"
published: true
category: en
tags: ['HTML5','Front-end']
cover: "http://ww4.sinaimg.cn/large/6d0af205jw1evtw72ji9pj20hs0aoq4b.jpg"
---

Generally, a file's type is identified by server side to check if this file is legal for storing. For instance, a cloud document editor can only allow users to import `.doc`/`.docx`/`.pages` files.

However, only detecting the expanded-name is not enough, some of users can change the file expanded-name to avoid this detection. Therefore, we have to find a solution to address this unsafe issue.

## Could a file's content can be modified ?

If we just modify the file's expanded-name, the file's content won't be changed. We can do an experiment. When we try to play a video file with a `.pdf` extension, it can still play well. In other words, the video player has another method to detect whether it is a video file or not. After searching on Google, I found that every file has a [File Signature (or Magic Number)](http://www.garykessler.net/library/file_sigs.html), which represents for the **real** type of a file. Fortunately, it is constantly embedded in the header of a file (first 4~8 bytes).

However, most solutions provided online are implementation on server side. It is not a good way to use. Only the file that has been uploaded to the server can be detected. A large file has to be waited for a long time to upload, which might make users mad. Therefore, I intended to find a front-end solution.

## How to detect file type through HTML5 ?

Thanks to [File API]() of HTML5, we can easily get the file signature of a file.

Here is the source code:

{% highlight html %}
<input type="file" onchange="checkFileType(this.files[0])"></input>

<script>
  function checkFileType(file){
    var slice = file.slice(0,4);      // Get the first 4 bytes of a file
    var reader = new FileReader();    // Create instance of file reader. It is asynchronous!
    reader.readAsArrayBuffer(slice);  // Read the chunk file and return to blob
    reader.onload = function(e) {
        var buffer = reader.result;          // The result ArrayBuffer
        var view = new DataView(buffer);      // Get access to the result bytes
        var signature = view.getUint32(0, false).toString(16);  // Read 4 bytes, big-endian，return hex string
        switch(signature) {                      // Every file has a unique signature, we can collect them and create a data lib.
          case "89504e47": file.verified_type = "image/png"; break;
          case "47494638": file.verified_type = "image/gif"; break;
          case "25504446": file.verified_type = "application/pdf"; break;
          case "504b0304": file.verified_type = "application/zip"; break;
        }
        console.log(file.name, file.verified_type);
  }
</script>
{% endhighlight %}

## In Practice

However, when I use it in practice, the file signature's length is not constant. Some of files only need first 4 bytes to detect, but others need first 8 bytes. Even worse, some files' signatures begin from the 512 bytes. How to use a universal solution to detect these different signatures ?

My current solution is that establishing a signatures library at first, and then using the expanded-name of files to match the signature of such kind of extension. If the signature can't match with the expanded-name, it will be considered illegally.

This is the signatures library I create:

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

Implementation:

{% highlight javascript %}
  // Get expanded-name of a file
  function getFileExtension(fileName) {
      var matches = fileName && fileName.match(/\.([^.]+)$/);
      if (matches) {
        return matches[1].toLowerCase();
      }
      return '';
  }

  // Detect if the expanded-name can match with the signature that belongs to it
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

## Summary

Detecting file type through front-end may be the fastest way, but there are also some cons.

### 1. Part of files has the same signatures

For instance, the signatures of files that Microsoft Office create (`.xlsx`,`.docx`,`.pptx`, etc.) are equal to that of zip files.

Test:

| File Type     |  Successfully Intercepted |
| :-------- | --------:|
| mp4 to pdf   | ✅ |
| zip to docx  | ✅ |
| zip to jpg   | ✅ |
| docx to zip  | ❌ |
| docx to xlsx | ❌ |

### 2. Compatibility

Unfortunately, HTML5's `FileReader` is not supported by all browsers.

| Feature   | Firefox (Gecko) | Chrome | Internet Explorer* | Opera* | Safari |
| :-------- | :-------------- | :----- | :----------------- | :----- | :----- |
| Basic support | 3.6 (1.9.2) | 7      | 10                 |  🚫    |  🚫   |


### 3. File Signature Library

The library of file signatures can be found on [File Signature Database](www.filesignatures.net), but not all the files has a signature on it, even though this website keeps updating them.

## Reference

[1] [FileReader API](https://developer.mozilla.org/zh-CN/docs/Web/API/FileReader)

[2] [Reading the first four bytes of a file](https://www.inkling.com/read/javascript-definitive-guide-david-flanagan-6th/chapter-22/reading-the-first-four-bytes-of)

[3] [FILE SIGNATURES TABLE](http://www.garykessler.net/library/file_sigs.html)

[4] [File Signature Database](http://www.filesignatures.net)
