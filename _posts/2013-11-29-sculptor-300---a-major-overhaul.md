---
layout: post
title: "Sculptor 3.0.0 - A Major Overhaul"
description: ""
category: 
tags: [Sculptor,Release,Xtext,Xtend,Eclipse,Maven,Git]
author: Sculptor Team
navbar_name: blog
---
{% include JB/setup %}

Sculptor 3.0.0 has been released. In this version Sculptor has undergone a major overhaul which was started at the beginning of this year.

This release has most of the functionality as previous 2.1.0, but internally it has been migrated to latest version of Eclipse4, Xtext2 and Xtend2. During this migration some of Sculptors features have been dropped (or haven't been migrated yet) - mainly the gui-related features.

On the other hand this version of Sculptor greatly benefits from the upgraded technology used internally, e.g.

* better usability of the DSL editor because of leveraging the latest version of Xtext and Eclipse
* better performance because all templates are now pure Java (compiled from Xtend code)
* better Maven support because Sculptor comes with its own [Maven plugin][2]

Now Sculptor is hosted on [GitHub](https://github.com/sculptor) and has its own [Website](http://sculptorgenerator.org).
[Sculptors Eclipse plugin][3] is available from [http://sculptorgenerator.org/updates](http://sculptorgenerator.org/updates) and the [Maven plugin][2] is available from [Maven Central](http://search.maven.org/#search%7Cga%7C1%7Cg%3A%22org.sculptorgenerator%22)


[Read more about the release][1].

   [1]: /documentation/whats-new#version-300
   [2]: /documentation/maven-plugin
   [3]: /documentation/eclipse-plugin
