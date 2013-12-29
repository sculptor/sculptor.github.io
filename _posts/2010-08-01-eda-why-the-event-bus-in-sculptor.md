---
layout: post
title: "EDA why the event bus in sculptor?"
description: ""
category: 
tags: [Sculptor,EDA,Scalability]
author: Andreas Källberg
navbar_name: blog
---
{% include JB/setup %}

We have come to our seventh entry about [Event Driven Architecture][1]. We are in the topic of [how Sculptor supports EDA][5]. In previous posts we have covered how to publish and subscribe, both through the dsl in the model, and through plain java code.

Today we will talk a bit more on the [thin layer we call 'the event bus'][5].

As stated before, there are a lot of different approaches to EDA. You can use/implement it locally or apply it too the entire enterprise. But also, in its simplest form its about the [observer pattern][4], regardless if we talk about big or small implementations. This leads us to the main motivations of our event bus abstraction.

We want to keep it simple, but at the same time still have the power of doing event handling big and small. In the Enterprise, or locally. And doing this with the same programming interfaces, i.e. keeping it simple.

So, based on the above we have the event bus api. And with the risk of repeating my self, it is a very simple one. Methods for publishing, subscribing, and un-subscribing.

It also ships with a default implementation named (you got it) simple event bus. This one is really easy (hey, I found another word instead of simple) to use and fully functional on its own. Though, if you need to integrate with another system you are going to need another implementation.

And that is one of our other motivation for the event bus, it should be easy to swap implementations.

Beside the default, we currently have two implementations based on [Spring Integration][2] and [Apache Camel][3]. Both of these are what's called "lightweight integration frameworks". Supporting these two frameworks brings a lot of power to the solution when it comes to integration.

And that brings us to the next topic of this series. Some examples of how to bring your events to life over system boundaries, i.e. integration stuff.


   [1]: http://en.wikipedia.org/wiki/Event-driven_architecture
   [2]: http://www.springsource.org/spring-integration
   [3]: http://camel.apache.org/
   [4]: http://en.wikipedia.org/wiki/Observer_pattern
   [5]: http://sculptorgenerator.org/2010/07/19/eda-sculptor-support/
