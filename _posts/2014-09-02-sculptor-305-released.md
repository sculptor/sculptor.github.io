---
layout: post
title: "Sculptor 3.0.5 Released"
description: ""
category: 
tags: [Sculptor,Release]
author: Sculptor Team
navbar_name: blog
---
{% include JB/setup %}

This maintainance release provides [a few bug fixes][2] and adds support for the Xtext / Xtend version 2.6.2. The generators internal workflow is migrated from a MWE2 script into a Xtend program and the generators built-in cartridges are refactored into separate Maven projects. The [Maven plugin][3] adds support for [Maven parallel builds][5].

[Read more about the release][1].

   [1]: /documentation/whats-new#version-305
   [2]: https://github.com/sculptor/sculptor/issues?q=milestone%3A3.0.5+is%3Aclosed
   [3]: /documentation/advanced-tutorial#cartridges
   [4]: /documentation/maven-plugin
   [5]: https://cwiki.apache.org/confluence/display/MAVEN/Parallel+builds+in+Maven+3
