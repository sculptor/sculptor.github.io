---
layout: post
title: "Sculptor 3.1.0 Released"
description: ""
category: 
tags: [Sculptor,Release,Xtext,Xtend,Eclipse]
author: Sculptor Team
navbar_name: blog
---
{% include JB/setup %}

In this release the technologies used or supported by Sculptor has been upgraded to newer versions:

* Eclipse was upgraded to version 4.4 ([Eclipse Luna](https://projects.eclipse.org/releases/luna))
* Xtext/Xtext was upgraded to [version 2.8.1](http://www.eclipse.org/Xtext/releasenotes.html)
* Spring was upgraded to version 4.1.6
* Hibernate was upgraded to version 4.2.18 (Hibernate 4.2 is the last version of Hibernate which does not require JPA 2.1)
* Jave EE support was upgraded to version 6

This release adds a first version of a Sculptor Cartridge for using [Spring Data JPA](http://projects.spring.io/spring-data-jpa/). By adopting Spring Data JPA Sculptors own JPA repository implementation ([generic access objects](http://sculptorgenerator.org/documentation/advanced-tutorial#generic-access-objects) and [generated finders](http://sculptorgenerator.org/documentation/advanced-tutorial#generation-of-finder-operations)) is replaced by the Spring Data framework (generic API across different persistence technologies). An introduction to Spring Data JPA can be found in Oliver Gierkes blog posts [Getting started with Spring Data JPA](http://spring.io/blog/2011/02/10/getting-started-with-spring-data-jpa) and [Advanced Spring Data JPA - Specifications and Querydsl](http://spring.io/blog/2011/04/26/advanced-spring-data-jpa-specifications-and-querydsl/).

A few new examples have been added:

* [springdatajpa-example](https://github.com/sculptor/sculptor/tree/master/sculptor-examples/springdatajpa-example) demonstrates the new Sculptor new cartridge for Spring Data JPA
* [springboot-example](https://github.com/sculptor/sculptor/tree/master/sculptor-examples/springboot-example) shows a Sculptor-based REST service with [Spring Boot](http://projects.spring.io/spring-boot/)
* [ejb-example](https://github.com/sculptor/sculptor/tree/master/sculptor-examples/ejb-example) contains the source code of Sculptors [Pure EJB3 Tutorial](http://sculptorgenerator.org/documentation/pure-ejb3-tutorial)

The following features have been dropped:

* Java 6 is not supported anymore (Sculptors is built with Java 7)
* JPA1 (with Hibernate 3) is not supported anymore - only JPA 2.0 (with Hibernate 4.2) is supported now

This release introduced the following API changes:

* Refactoring of `ServiceContext` into package `org.sculptor.framework.context`
* Refactoring of `JpaFlushEager` into package `org.sculptor.framework.persistence`

So check out the new version of Sculptors [Eclipse plugin][2] (available from [http://sculptorgenerator.org/updates](http://sculptorgenerator.org/updates)) and the [Maven plugin][3] (available from [Maven Central](http://search.maven.org/#search%7Cga%7C1%7Cg%3A%22org.sculptorgenerator%22)).


[Read more about the release][1].

   [1]: /documentation/whats-new#version-310
   [2]: /documentation/eclipse-plugin
   [3]: /documentation/maven-plugin
