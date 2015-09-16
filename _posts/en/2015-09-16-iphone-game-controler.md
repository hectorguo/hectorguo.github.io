---
layout: post
title: Use iPhone to build a Remote Game Controller just like Wii
categories: ['en']
tags: ['html5']
published: True
cover: 'http://ww3.sinaimg.cn/large/6d0af205jw1ew4mio6htfj20go09gwgt.jpg'
---

Do you like playing motion-sensing games like this? Have you ever thought that it might be prohibitively expensive to play such a cool game? Actually, you can use your mobile phone to play just like that. Below are a demo.

<video poster="//i.imgur.com/hDSmdI9h.jpg" preload="auto" autoplay="autoplay" muted="muted" loop="loop" webkit-playsinline="" style="width: 300px; height: 360px;">
    <source src="//i.imgur.com/hDSmdI9.webm" type="video/webm">
    <source src="//i.imgur.com/hDSmdI9.mp4" type="video/mp4">
</video>

If you want to use your phone to play games like above, you can just follow these steps below, which is a easy approach.

## Prerequisite
1. iPhone or other phones with Accelerometer embedded.
2. PC (make sure [nodejs](https://nodejs.org/), [express](http://expressjs.com/)  and [socket.io](http://socket.io/) have been installed).
3. Chrome Browser or other browsers that support [HTML5](www.w3schools.com/html/html5_intro.asp).

## Device Motion Intro
As we can see, HTML5 provide some apis for device motions, these apis can help us to identify our device's orientation and movements. And we can use them to create some amazing apps such as Compass and Pedometer. Also, we can use them to create a remote Game Controller like Wii. 
![](http://dab1nmslvvntp.cloudfront.net/wp-content/uploads/2014/05/1398214919axes.png)
The device motion event fires on a regular interval and returns data about the rotation (in ° per second) and acceleration (in m per second2) of the device, at that moment in time. Some devices do not have the hardware to exclude the effect of gravity.

The event returns four properties, `accelerationIncludingGravity`, `acceleration`, which excludes the effects of gravity, `rotationRate` and `interval`.

For example, let’s take a look at a phone, lying on a flat table, with its screen facing up.

| State | Rotation | Acceleration(m/s2) | Acceleration with gravity (m/s2) |
| ------------- | ------------- | ---------| ----------------------------- |
| Not moving | [0, 0, 0] | [0, 0, 0] | [0, 0, 9.8] |
| Moving up towards the sky | [0, 0, 0] | [0, 0, 5] | [0, 0, 14.81] |
| Moving only to the right | [0, 0, 0] | [3, 0, 0] | [3, 0, 9.81] |
| Moving up & to the right | [0, 0, 0] | [5, 0, 5] | [5, 0, 14.81] |

Conversely, if the phone were held so the screen was perpendicular to the ground, and was directly visible to the viewer:

| State | Rotation | Acceleration(m/s2) | Acceleration with gravity (m/s2) |
| ------------- | ------------- | ---------| ----------------------------- |
| Not moving | [0, 0, 0] | [0, 0, 0] | [0, 9.81, 0] |
| Moving up towards the sky | [0, 0, 0] | [0, 5, 0] | [0, 14.81, 0] |
| Moving only to the right | [0, 0, 0] | [3, 0, 0] | [3, 9.81, 0] |
| Moving up & to the right | [0, 0, 0] | [5, 5, 0] | [5, 14.81, 0] |

Therefore, we can use `acceleration` to simulate four basic directions: `UP`, `DOWN`, `LEFT`, `RIGHT`

{% highlight javascript %}

{% endhighlight %}

## Send Messages to PC
We can utilize [socket.io](http://socket.io/) to keep a connection between PC and iPhone. By doing this, the messages sent from iPhone can be received by PC, which is the key to simulate a remote Game Controller with iPhone.

### Server Side:

{% highlight javascript %}
var express = require('express');
var app = express();
var http = require('http').Server(app);
var io = require('socket.io')(http);

app.use(express.static(__dirname));

// send messages to Browser on PC once receiving them from devices
io.on('connection', function(socket){
  socket.on('direction', function(msg){
    io.emit('direction', msg);
  });
});

// start a server
http.listen(3000, function(){
  console.log('listening on *:3000');
});
{% endhighlight %}

### Client Side:

mobile.html (loaded by iPhone):

{% highlight html %}
<script src="/socket.io/socket.io.js"></script>
<script>
// send direction info to server side
function infoSend(direction){
  var socket = io();

  setTimeout(function(){  // fix some incorrect directions when you move the device
    count--;
    if(count === 0){
      socket.emit('direction', direction);
    }
  }, 100);
  count++;
}

// detect the direction of device
function deviceMotionHandler(event){
  var acceleration = event.acceleration;

  if(acceleration.x > 10) {
    infoSend('RIGHT');
  } else if(acceleration.x < -10) {
    infoSend('LEFT');
  }

  if(acceleration.y > 10) {
    infoSend('UP');
  } else if (acceleration.y < -10) {
    infoSend('DOWN');
  }

}

if (window.DeviceMotionEvent) {
  var count = 0;
  window.addEventListener('devicemotion', deviceMotionHandler, false);
}
</script>
{% endhighlight %}

browser.html (loaded by Browser on PC such as Chrome) :

{% highlight html %}
<script src="/socket.io/socket.io.js"></script>
<script>
var socket = io();
  socket.on('direction', function(msg){
    console.log(msg);
    var keyCode;
    switch (msg){
      case 'UP':
        keyCode = 38;
        break;
      case 'RIGHT':
        keyCode = 39;
        break;
      case 'DOWN':
        keyCode = 40;
        break;
      case 'LEFT':
        keyCode = 37;
        break;
    }
    snaky.move({keyCode:keyCode}); // simulate arrow keypress and make the snake move
  })
</script>
{% endhighlight %}

Now, all the fundamental work has been done. The rest is just to add some UI to respond to users.
This demo has been hold on [Github](https://github.com/hectorguo/iPhone-remote-game-controller) , you can fork it to make your own funny games.

## Rotation Data

HTML5 also provide some Rotation-related apis, here are some basic instructions. You can visit [this page](http://www.sitepoint.com/using-device-orientation-html5/) to get more details.

### Alpha
The rotation around the z axis and is 0° when the top of the device is pointed directly north. As the device is rotated counter-clockwise the `alpha` value increases.
![](http://dab1nmslvvntp.cloudfront.net/wp-content/uploads/2014/05/1398214917alpha.png)

### Beta
The rotation around the x axis and is 0° when the top and bottom of the device are equidistant from the surface of the earth. The value increases as the top of the device is tipped toward the surface of the earth.
![](http://dab1nmslvvntp.cloudfront.net/wp-content/uploads/2014/05/1398214915beta.png)

### Gamma
The rotation around the y axis and is 0° when the left and right of the device are equidistant from the surface of the earth. The value increases as the the right side is tipped towards the surface of the earth.
![](http://dab1nmslvvntp.cloudfront.net/wp-content/uploads/2014/05/1398214914gamma.png)

## Reference

[1] [Device motion](https://developers.google.com/web/fundamentals/device-access/device-orientation/dev-motion?hl=en)

[2] [Using Device Orientation in HTML5](http://www.sitepoint.com/using-device-orientation-html5/) 

[3] [Snakylines Game powered by HTML5](https://github.com/liege/Snakylines/blob/master/Snakylines-v3.0.html) 
