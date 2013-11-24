---
layout: post
title: "Pick 'n' Choose Target Implementation"
description: ""
category: 
tags: [Sculptor, Customization]
author: Sculptor Team
navbar_name: blog
---
{% include JB/setup %}

This article gives an overview of the options you have regarding the generated target implementation. We add new stuff all the time so this describes what is available at the time of this writing, i.e. in Sculptor 1.7.0.

The default target implementation is using:


  * JPA with Hibernate as provider
  * Spring with annotations, no EJBs
  * Web CRUD GUI client with JSF, Facelets, and Spring Web Flow
  * In-memory Hsqldb as database
  * Deployment as war in Jetty

The default choices gives a good starting environment with minimal requirements of installation of external software, such as database and application server.

For each of these areas there are alternative implementations that are supported by Sculptor out of the box. The features are selected with simple configuration.

![Sculptor target implementation][1] \\
<small>_Figure 1. Sculptor target implementation_</small>

If you have a different reference architecture than what is provided by Sculptor out of the box you can rather easily customize the generator to fit your needs. It is possible to add your own code generator templates or redefine sections of existing templates. All that is described in the [Developers Guide][2].

   [1]: /images/2010-01-16-pick-n-choose-target-implementation/SculptorTargetImplementation.png
   [2]: /documentation/developers-guide