---
layout: post
title: "Sculptor 1.7.0 - GAE, Smartclient, EclipseLink, DataNucleus, JEE"
description: ""
category: 
tags: [Sculptor,Release]
author: Sculptor Team
navbar_name: blog
---
{% include JB/setup %}

Sculptor 1.7.0 has been released. In this version the list of supported technologies has grown with several popular alternatives.

Sculptor has many built in customization options to[ pick and choose from][1]. The major new target implementation options in this release:

  * Google App Engine
  * EclipseLink and DataNucleus JPA Provider
  * Smartclient GWT
  * Pure EJB3 (without Spring)
  * Web Services with JAX-WS

Sculptor also has good possibilities for extensibility if you prefer a design or framework that is not supported out-of-the-box. Sculptor is using openArchitectureWare code generation platform, which makes it possible to change the code generation templates using Aspect-Oriented Programming (AOP) features.

The Pure EJB3 implementation, one of the new features in 1.7.0, has already been successfully used in several projects, i.e. it is well tested and production ready. This means that not only Spring users can gain the productivity and quality benefits of Sculptor, but also those who develop on the standard JEE stack.

[Read more about the release][2].

   [1]: /2010/01/16/pick-n-choose-target-implementation
   [2]: /documentation/whats-new#version-170
