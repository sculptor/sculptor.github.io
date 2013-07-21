---
layout: page
title: "DDD Sample"
description: ""
navbar_name: docs
---
{% include JB/setup %}


This page describes a port of the [DDD Sample][1] using Sculptor. It is a domain model based on the cargo example used in [Eric Evans' book][2].

Sculptor will help you with the static structure of the application. It can generate the data part of the domain objects and
boilerplate code for services and repositories. It has a nice runtime framework for especially the persistence implementations,
such as standard queries. However, to qualify as a true DDD application the domain objects must have complex behavior - business logic.
With this sample we illustrate how to integrate manually written logic with the automatically generated parts.

**Table of Contents:**

* toc
{:toc}


## Model

The definition of the domain model, using Sculptor's textual DSL is separated in one file for each Module, plus one "empty" top file, which imports the other files.

The generated vizualization with [GraphViz][3] looks like this:
![Graphviz visualization][4]


## JUnit Tests

This sample has an extensive test suite, which illustrates how to write junit tests at different levels.

  * logic in the domain objects is tested with ordinary junit tests, without any need for database emulation or spring container
  * repositories and details of the persistence are tested with DBUnit tests and spring container and hibernate using in memory database
  * services are tested in a similar way as the repositories, but also using easymock framework


## Metrics

It is interesting to compare the fully hand written original [DDD Sample][1] with this partly generated port. The number of hand written lines of code in the Sculptor port is 783. Compared to 1179 in the original. The business logic is almost identical, but the Sculptor variant has less lines of code since much of the boring boilerplate code is generated. It has bean measured with [JavaNCSS][5].
![Lines of code][6]

![Lines of code - per package][7]

![Lines of code - summary][8]

JUnit tests are not included, since they are very similar. Web application is not included, since it is not implemented in this Sculptor port.


## Try It

If you are only interested in a sneak preview of the source code you can download it here: [Sculptor-DDDSample-src.zip][9]

Another choice is to try the real thing:

 1. Install Sculptor and its requisites as described in the [Installation Guide][10].
 2. Get the source code from GitHub: 

    ~~~
    $ git clone git://github.com/sculptor/sculptor.git
    $ cd sculptor/sculptor-example/DDDSample
    ~~~

 3. Build with `mvn clean install`. This will also run all JUnit tests located in `src/test/java`. Take a look at some of them and run them from Eclipse also.
 4. Study the `.btdesign` files located in `src/main/resources`.
 5. Study the hand written code in `src/main/java` and the generated code in `src/generated/java`.

Learn more about the capabilities of Sculptor by reading the [Hello World][11] and [Advanced Tutorial][12]


## Source Code

The complete source code for this sample is available in GitHub [https://github.com/sculptor/sculptor/tree/master/sculptor-example/DDDSample](https://github.com/sculptor/sculptor/tree/master/sculptor-example/DDDSample).


   [1]: http://dddsample.sourceforge.net/
   [2]: http://www.domaindrivendesign.org/books/index.html#DDD
   [3]: http://www.graphviz.org/
   [4]: /images/documentation/ddd-sample/ddd-sample-model.png
   [5]: http://www.kclee.de/clemens/java/javancss/
   [6]: /images/documentation/ddd-sample/ddd-sample-loc-bar.png
   [7]: /images/documentation/ddd-sample/ddd-sample-loc-package.png
   [8]: /images/documentation/ddd-sample/ddd-sample-loc-summary.png
   [9]: /images/documentation/ddd-sample/Sculptor-DDDSample-src.zip
   [10]: installation
   [11]: hello-world-tutorial
   [12]: advanced-tutorial
  
