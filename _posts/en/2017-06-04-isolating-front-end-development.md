---
layout: post
title: "Two practices for isolating front-end development from back-end"
categories: ['en']
tags: ['Front-end', 'ES6']
cover: "https://raw.githubusercontent.com/hectorguo/blog-imgs/master/img/20190608214346.png"
published: True
---

It is not uncommon that when a project is initialized, engineering team starts to build front-end and back-end simultaneously. Sometimes, there are some dependencies with each other — UI-side needs data from server-side for validation, Server-side needs an interface for testing. However, we can not do nothing until another side’s ready. Below are two practices I want to share for keeping front-end development separate from back-end.

# 1. Host a local server

![JSONPlaceholder](https://raw.githubusercontent.com/hectorguo/blog-imgs/master/img/20190608214357.png)

Before waiting for server-side being ready, we can communicate with back-end team to come to an agreement that how the APIs and data format looks like. With these APIs, we can easily create some fake data and host a local server for testing. I recommend JSONPlaceholder, a powerful tool for faking Online REST API. It supports all HTTP Methods, and after you POST a data to the server, the data will also be stored in your local machine, which looks like a real web services. Moreover, it also provide a remote service which allow you to test without hosting a local server.
In the front-end code, it can be like this:

{% highlight javascript %}
// Replace this URL when server-side is ready
const SERVER_URL = 'https://jsonplaceholder.typicode.com';
export function getUsers() {
   return fetch(`{SERVER_URL}/users`)
          .then(res => res.json());
}
...
// Use the data
getUsers()
.then((users) => {
   // render users to DOM
});
{% endhighlight %}

By doing this, you don’t need to change lots of code when the server-side is ready. Just replace the SERVER_URL, the real data will be shown correctly.

# 2. Leverage Promise

![Promise](https://raw.githubusercontent.com/hectorguo/blog-imgs/master/img/20190608214414.png)

Sometimes, hosting a local server is not practical for sharing. For example, a Product Manager or Test team need to see the UI-side earlier, you can not expect the PM or test team to install a local server on their machine each by each. Also, it is not easy to host a fake server on the internal development machine which is shared by teams. Therefore, we can fake the data in our codes like this:

{% highlight javascript %}
// const SERVER_URL = 'https://realhost';
export function getUsers() {
   // return fetch(`{SERVER_URL}/users`);
  
  // Fake data in the code
  return new Promise(resolve, reject) => {
    const users = [{
        uid: "1",
        first_name: "hector",
        last_name: "guo"
    },{
        uid: "2",
        first_name: "john",
        last_name: "lee"
    }]
    resolve(users);
   });
}
...
// Use the data
getUsers()
.then((users) => {
   // render users to DOM
});
{% endhighlight %}

In this way, we are leveraging Promise to transfer the sync pattern to async pattern. By doing this, we don’t need a local or remote server to run our UI. Just share a static html file with PMs or test team, they can see the result immediately. After the server-side is ready, all we need to do is just remove / comment the fake data code, and uncomment the fetch from server-side.
If there are more practical practices you want to share, please don’t hesitate to comment.