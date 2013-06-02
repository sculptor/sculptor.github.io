---
layout: page
title: Overview
navbar_name: docs
wide: true
hide_header: true
---
<div class="row">
  <div class="sidebar span3">
    <ul id="sidenav" class="nav nav-list affix">
      <li class="active"><a href="#overview">Introduction</a></li>
      <li><a href="#strengths">Strengths</a></li>
      <li><a href="#license">License</a></li>
      <li><a href="#issues-and-mprovements">Issues and Improvements</a></li>
    </ul>
  </div>
  <div class="span9">
    <div class="page-header">
      <h1>{{ page.title }}</h1>
    </div>
    <div markdown="1">

![Sculptor Banner](/images/sculptor-banner.png)

## Introduction

Sculptor is an open source productivity tool that applies the concepts from [Domain-Driven Design](http://domaindrivendesign.org/books/) and Domain Specific Languages.

You express your design intent in a textual specification, from which Sculptor generates high quality Java code and configuration. You can use the concepts from Domain-Driven Design (DDD) in the textual Domain Specific Language (DSL). E.g. Service, Module, Entity, Value Object, Repository.

The generated code is based on well-known frameworks, such as [JPA](http://java.sun.com/javaee/technologies/persistence.jsp), [Hibernate](http://www.hibernate.org/), [Spring Framework](http://www.springframework.org/), [Spring Web Flow](http://www.springframework.org/webflow), [JSF](http://java.sun.com/javaee/javaserverfaces/) and [Java EE](http://java.sun.com/javaee/). Sculptor takes care of the technical details, the tedious repetitive work, and let you focus on delivering more business value - and have more fun.

The DSL and the code generation drives the development and is not a one time shot. The application can be developed incrementally with an efficient round trip loop.

Sculptor is useful when developing typical enterprise or web applications that benefit from a rich and persistent domain model. Sculptor also provides a sophisticated CRUD GUI for administrative sections of the application or to serve as a scaffolding for your manually created pages.

Within 15 minutes you can go from scratch to a running application, including build scripts, Eclipse projects, domain model, JPA persistence, services, Web Flow application and much more. Thereafter you can continue by evolving the design, add manual code and regenerate.

Sculptor is not an one-size-fits-all product. Even though it is a good start for many systems, sooner or later customization is always needed. Sculptor is designed and documented with this in mind. The generated result can easily be modified to meet your needs.


## Strengths

* Easy to learn, intuitive syntax of the textual DSL, based on the concepts from DDD
* Textual DSL has a lot of productivity benefits over graphical tools
* [Quick start](http://fornax-sculptor.blogspot.com/2009/06/jump-start.html), simple installation
* Quick development round trip, short feedback loop, it is not a one time shot
* Support for [TDD](http://fornax-sculptor.blogspot.com/2009/08/testing-is-simple.html) and [refactoring](http://fornax-sculptor.blogspot.com/2009/08/refactoring.html)
* Existing IDE tools, such as refactoring, code assist and debugger will continue to be of service to you
* [High quality](http://fornax-sculptor.blogspot.com/2009/11/promote-quality-with-sculptor.html) of generated code
* [Pick 'n' Choose Target Implementation](http://fornax-sculptor.blogspot.com/2010/01/pick-n-choose-target-implementation.html), based on well known frameworks, best practices, and a lot of experience
* Generation of complete application from a single model, not only fragments that are hard to fit in to the overall design
* Great extensibility and customization
* Used with de facto standard build tool - Maven
* Based on [Xtext](http://www.eclipse.org/Xtext/) code generation framework
* Can be used with text editor or any IDE, but DSL editor with error highlight, code completion, and outline is provided for Eclipse users
* Easy to [remove the tool](http://fornax-sculptor.blogspot.com/2010/01/how-to-remove-sculptor.html), no runtime magic


## License

Sculptor is licensed under the [Apache 2.0 License](http://www.apache.org/licenses/LICENSE-2.0).


## Issues and Improvements

Use the [Forum](http://groups.google.com/group/fornax-platform) for questions and discussion.

You can report bugs and feature request in the [forum](http://groups.google.com/group/fornax-platform) also. We will add them to issue tracking system: [https://github.com/sculptor/sculptor/issues](https://github.com/sculptor/sculptor/issues).

  </div>
</div>
