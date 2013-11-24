---
layout: post
title: "EDA in Massive Web Sites"
description: ""
category: 
tags: [EDA, Scalability]
author: Patrik Nordwall
navbar_name: blog
---
{% include JB/setup %}

This is a comment to [@h3nk3][1]'s latest blog post on [Concurrency, Parallelism and Actors][2].

I totally agree that there is no single solution for everything. Regarding massive web sites I would like to add a few things to your conclusions, which makes asynchronous event driven solutions important for web sites also.

You should try to do as little as possible in the request thread, i.e. minimize latency. Threads in them selves are a scarce resource. You can do the essential validation and then hand off the rest of the work to a task queue (e.g. Flickr). You can precompute everything and cache it so that serving the requests becomes trivial (e.g. Reddit). Both these techniques can benefit from using event driven solutions in the backend.

We see great improvements in the area of truly asynchronous web solutions, which means that you don't have to deliver a synchronous response to every request. The result can be pushed to the clients asynchronously. Then it becomes natural to use event driven solutions in the server. The reasons for using Actors it is not only scalability and concurrency, it is also much about using a simpler concurrency model than threads and locks.

Of course it is also important to use HTTP as designed, since HTTP caching is a key ingredient in highly scalable web sites.

   [1]: http://twitter.com/h3nk3
   [2]: http://r-c-r.tumblr.com/post/2947120096/concurrency-parallelism-and-actors
