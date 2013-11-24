---
layout: post
title: "Sculptor 2.0 is out"
description: ""
category: 
tags: [Sculptor, Release]
author: Sculptor Team
navbar_name: blog
---
{% include JB/setup %}

This is the 11th major release of Sculptor.

Three new and noteworthy features:

1. The new [REST support][1] in Sculptor makes it very easy to expose restful services by using conventions and easy delegation to Services to reduce boilerplate coding. [Spring MVC][2] is the underlaying REST framework.

2. [Mixin composition][3] is something Java developers are missing, but with Sculptor it becomes available.

3. The services can now easily be used with [Spring Remoting][4]. This is convenient for the [RCP client][5], but can be used for other clients also.

In addition to these and several other new features we have also improved the technical quality.

We have upgraded support for latest versions of most of the underlaying tools and frameworks. Most important is Xtext 1.0, Eclipse Helios, Maven 3, Spring 3.0.

The logging API has been changed from Commons Logging with Log4j to [SLF4J][6] with [Logback][7]. SLF4J allow many backends and also have special bridges for many (legacy) logging frameworks. SLF4J with Logback is considered to be best logging framework available.


   [1]: /documentation/rest-tutorial
   [2]: http://static.springsource.org/spring/docs/3.0.x/reference/mvc.html
   [3]: /2011/02/03/mixin-composition
   [4]: http://static.springsource.org/spring/docs/3.0.x/spring-framework-reference/html/remoting.html
   [5]: http://fornax.itemis.de/confluence/x/zQk
   [6]: http://www.slf4j.org/
   [7]: http://logback.qos.ch/