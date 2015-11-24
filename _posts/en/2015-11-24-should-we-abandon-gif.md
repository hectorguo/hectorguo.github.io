---
layout: post
title: Should We Abandon GIF ?
categories: ['en']
tags: ['Media Tech']
published: True
cover: http://ww1.sinaimg.cn/large/6d0af205gw1eycdd5zefhj20hd09swfx.jpg
---

Recently, I captured an [interesting video](http://hectorguo.com/en/iphone-game-controler/) and prepared to share with my friends. I wondered which way I should choose to share it online, uploading to Youtube, or just converting it into a GIF ?  Given that gif is really common and more compatible for a variety of devices than video, so I upload it into [Imgur](http://imgur.com/), the most popular gif distributing platform. Unfortunately, after converting my video into gif, I found the gif format was even much larger than my video. It did astonish me. If gif's size is larger than a video, why not use video ?  Afterall, it can load faster and cost lower than gif.

> I found the gif format was even much larger than my video.

Therefore, I start to search for the best practice in this case. I find that imgur's website have replaced gif with video for a long time, which is called as [GIFV](https://imgur.com/blog/2014/10/09/introducing-gifv/), using HTML5 `<video>` to load animation images. The advantage of this technology is that a video can be compressed more than a gif does, and it runs more smooth than gif. Because it can be encoded with H264, H265 or VP9, which is an Advanced Video Coding. It sounds interesting. In comparison with gif, I am willing to see how the actual benefit of video is. Let's convert my video into VP9 format that is the latest coding standard.

## Install FFMPEG
Af first, we have to install the encoder of VP9 to finish our task. Here is the [offical guide](https://trac.ffmpeg.org/wiki/CompilationGuide/Ubuntu). But it is really hard to install because of lots of dependencies. And here is the easiest way I think to install all of them by using [`Homebrew`](http://brew.sh/).

```
brew install ffmpeg --with-libvpx
```

And if you don't have installed `Homebrew` yet, here is the quick way:

```
ruby -e "$(curl -fsSL https://raw.github.com/mxcl/homebrew/go)"
```

## Convert video to VP9 format
I admit that there are lots of options in `ffmpeg` command. If you just want to see the result, here is the common use:

```
# Convert a.mp4 to output.webm without audio channel
ffmpeg -i a.mp4 -c:v libvpx-vp9 -threads 8 -speed 1 -tile-columns 6 -frame-parallel 1 -auto-alt-ref 1 -lag-in-frames 25 -an -strict experimental output.webm
```

- `-i a.mp4` means to get the source `a.mp4`
- `-c:v libvpx-vp9` means to encode video with vp9
- `-threads 8 -speed 1 -tile-columns 6 -frame-parallel 1` all these options are used to optimize efficiency of encoding and decoding
- `-an` means to eliminate the audio channel (to eliminate video channel, you can use `-vn`)
- `-strict experimental` means to use experimental encoder that has not been supported such as `libopus`

For more information about them, you can visit [vp9-encoding-guide](http://wiki.webmproject.org/ffmpeg/vp9-encoding-guide).

## Comparison
After a few hours, I have successfully converted my videos to VP9 format. Here is the size comparison between VP9 video and gif.

![](http://ww3.sinaimg.cn/large/6d0af205gw1eyccckkzuyj20bm01tgll.jpg)

Here is a comparison between VP9 format and H264 format:

![](http://ww4.sinaimg.cn/large/6d0af205gw1eyccby0j55j20c80arjs1.jpg)

You can see, the quality of video aren't reduced even though the size of VP9 is 67% less than that of H264.

![](http://ww2.sinaimg.cn/large/6d0af205gw1eycccso257j20b9019q2z.jpg)

![](http://ww2.sinaimg.cn/large/6d0af205gw1eyccgin81oj21960yeana.jpg)

## Summary
After viewing the result above, I think it is time to abandon GIF, even though it is really convenient to share on the web platform. Maybe you don't notice that most of media websites (Twitter, Facebook, Imgur, gyfcat, etc.) have used video instead of gif already.

After this practice, the reason why I can still view a HD video on Youtube even in a poor speed may have been found by myself. 😊
