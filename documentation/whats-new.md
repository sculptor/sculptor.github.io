---
layout: page
title: "What's new in Sculptor"
description: "Release notes"
navbar_name: docs
---
{% include JB/setup %}

Released versions of Sculptor:

* toc
{:toc}

## Version 3.0.x

### Version 3.0.3

Version 3.0.3 was released March xx, 2014.

This release contains:

* Support for [generator properties](developers-guide#generator-properties) which are common within a [multi-module Maven project](http://maven.apache.org/guides/mini/guide-multiple-modules.html)
* Refactoring of [Maven plugins](maven-plugin) status file handling to improve file deletion on Windows
* Removal of obsolete code related to partial generation and CRUD UI generation 


### Version 3.0.2

[Version 3.0.2 was released March 2, 2014](/2014/03/02/sculptor-302-released).

This release contains:

* Upgrade to [Xtext/Xtend 2.5.3](http://www.eclipse.org/Xtext/releasenotes.html)
* Refactoring of Sculptor generator test code into separate Maven module
* Removal of Fornax OAW Maven plugin from build (Exec Maven plugin is used now)


### Version 3.0.1

[Version 3.0.1 was released January 18, 2014](/2014/01/18/sculptor-301-released).

This release contains:

* Upgrade to [Xtext/Xtend 2.5.0](http://www.eclipse.org/Xtext/releasenotes.html)
* Support for [mongoDB](http://mongodb.org) in [Maven archetypes](maven-archetypes#sculptor-maven-archetype)
* Support for [Maven 3.1 logging](http://maven.apache.org/maven-logging.html) in [Maven plugin](maven-plugin)
* Support for Eclipse projects with [Maven Integration (m2e)](http://www.eclipse.org/m2e/) nature ([resource folders in Java project have `excluded="*"` set in classpath properties](http://wiki.eclipse.org/M2E_FAQ#Why_resource_folders_in_Java_project_have_excluded.3D.22.2A.22))
* Additional validations and quickfixes, e.g. for repositories in non-persistent events or reserved keywords used as ID
* Support for hyperlink navigation to imported model files
* Custom icon for [Eclipse text editor](eclipse-plugin) and feature branding
* Additional [EDA examples](https://github.com/sculptor/sculptor/tree/develop/sculptor-examples/eda-samples) and missing [MongoDB examples](https://github.com/sculptor/sculptor/tree/develop/sculptor-examples/mongodb-samples)


### Version 3.0.0

[Version 3.0.0 was released November 29, 2013](/2013/11/29/sculptor-300---a-major-overhaul).

Some highlights of the release:

* Upgrade to [Xtext/Xtend 2.4.3](http://www.eclipse.org/Xtext/releasenotes.html) and Eclipse Kepler
* Java package changed from `org.fornax.cartridges.sculptor` to `org.sculptor`
* New Home Page: [http://sculptorgenerator.org](http://sculptorgenerator.org)
* New Forum: [https://groups.google.com/group/sculptorgenerator](https://groups.google.com/group/sculptorgenerator)
* Version control changed from Subversion to Git
* Project hosting changed from [Sourceforge](http://sourceforge.net/projects/fornax/) to [GitHub](https://github.com/sculptor)
* Website implemented via [Jekyll Bootstrap](http://jekyllbootstrap.com)
* [Maven plugin](maven-plugin) available from [Maven Central](http://search.maven.org/#search%7Cga%7C1%7Cg%3A%22org.sculptorgenerator%22)
* [Eclipse plugin](eclipse-plugin) available from [http://sculptorgenerator.org/updates](http://sculptorgenerator.org/updates)
* Eclipse plugin build with Maven via [Eclipse Tycho](http://eclipse.org/tycho/)
* Release process based on [Gitflow Maven plugin](https://bitbucket.org/atlassian/maven-jgitflow-plugin)
* CI service via [Travis CI](https://travis-ci.org)


## Version 2.1.x

### Version 2.1.0

[Version 2.1.0 was released January 21, 2012](/2012/01/21/sculptor-210-released).

Some highlights of the release:

* JPA 2.0
* Convenient ways to query the database by defining queries in the model, see documentation in [Advanced Tutorial][1]
* Support for OpenJPA
* Validation based javax.validation api
* Builders with fluent interface for construction of domain objects, see documentation in [Advanced Tutorial][1]
* Major performance improvement in generation by suppressing invisible logging in beautifier


## Version 2.0.x

### Version 2.0.0

[Version 2.0.0 was released March 13, 2011](/2011/03/13/sculptor-20-is-out).

This release contains:

* RESTful Services
* Mixin composition with traits
* Spring Remoting
* Upgrade to Xtext 1.0 and Eclipse Helios
* Upgrade to Maven 3
* Upgrade to Spring 3.0
* Upgrade of versions for Google App Engine, Apache Camel, Spring Integration, Spring Web Flow, Drools, MongoDB, JBoss, Ehcache, Joda
* Change of logging API to SLF4J
* Removal of deprecated features
* Support DataTransferObject in CRUD GUI
* Persistent DomainEvent


## Version 1.9.x

### Version 1.9.0

[Version 1.9.0 was released July 07, 2010](/2010/07/07/sculptor-190---support-for-mongodb-and-event-driven-architecture).

This release contains:

* Support for MongoDB
* Support for Event-Driven Arcitecture (Publish/Subscribe, CQRS, Event Sourcing)
* New Home Page: [http://sculptor.fornax-platform.org](http://sculptor.fornax-platform.org)
* Generated Documentation
* More generated visualizations
* DSL language syntax
* GWT SmartClient improvements
* Support for Non persistent ValueObject in GUI
* Several improvements of generation of database DDL
* ToStringStyle


## Version 1.8.x

### Version 1.8.0

[Version 1.8.0 was released February 16, 2010](/2010/02/16/sculptor-180-released).

This release has the same functionality as previous 1.7.0, but internally it has been migrated to latest openArchitectureWare, which is now part of Eclipse. This means that the usability of the DSL editor is much better, e.g

* Better content assist (ctrl+space)
* Formatting (Pretty Printing) of model files (ctrl+shift+F)
* Templates (ctrl+space). This is a good complement to ordinary content assist. For example collection references.


## Version 1.7.x

### Version 1.7.0

[Version 1.7.0 was released January 29, 2010](/2010/01/29/sculptor-170---gae-smartclient-eclipselink-datanucleus-jee).

This release contains:

* Support Google App Engine
* Support for additional JPA providers, EclipseLink, and DataNucleus
* CRUD GUI with GWT SmartClient
* JSF improvements
* Pure EJB3 target implementation (without Spring)
* Data Transfer Objects and Web Services with JAX-WS
* Pagination for queries
* Complex criteria support with internal DSL
* findByKey, built in query for the properties marked as key
* Hint for customization


## Version 1.6.x

### Version 1.6.0

Version 1.6.0 was released June 14, 2009.

This release contains:

* Support for JPA
* Support for Hibernate validator
* Spring annotations instead of XML
* JUnit 4 annotations
* Jetty and inmemory Hsqldb
* Spring Webflow 2
* Don't generate gap classes by default


## Version 1.5.x

### Version 1.5.0

Version 1.5.0 was released February 1, 2009.

This release contains:

* New DSL for customization of the CRUD GUI
* New rich client dialect of the CRUD GUI
* Upgrade to oAW 4.3.1
* Port of the DDD Sample


## Version 1.4.x

### Version 1.4.1

Version 1.4.1 was released October 6, 2008.

The most important features of this release:

* JSF dialect of CRUD GUI.
* Possibility to split model.design into several files.
* Upgrade to oAW 4.3
* Support for Eclipse 3.4 Ganymede. Europa 3.3.2 is also supported.


## Version 1.3.x

### Version 1.3.0

Version 1.3.0 was released March 17, 2008.

The most important feature of 1.3 is a totally new metamodel for the CRUD GUI. In version 1.1 we implemented the CRUD GUI without a separate gui metamodel. The gui generation was using the business tier model. This was a good start, but the experience was that the templates and helper extensions became rather complicated. A special purpose gui metamodel simplified the templates and moved some of the logic to a transformation instead. This is important as we see the need for different dialects of the GUI. In the initial version we are using Spring WebFlow and JSP. We have also started a JSF dialect.

More is likely to come, such as rich client (RCP). Another motivation for the gui metamodel is the possibility to customize the GUI by using a separate DSL (in next release).


### Version 1.3.1

Version 1.3.1 was released April 11, 2008.

This release is a minor bug fix release.


## Version 1.2.x

### Version 1.2.0

Version 1.2.0 was released December 9, 2007.

It is a technical upgrade of several core products such as Eclipse and openArchitectureWare.


## Version 1.1.x

### Version 1.1.0

Version 1.1.0 was released November 5, 2007.

This release includes a lot of useful features:

* A complete, but customizable, CRUD Gui is generated based on the domain model specified with the Sculptor DSL. The generated web application uses Spring Web Flow to achieve navigation of the associations in the domain model.
* Scaffold simplifies generation of CRUD operations and is perfect for prototyping together with the CRUD Gui.
* It is efficient to develop the domain model with a textual DSL, but for communication purposes it is useful to have a visual UML view. Sculptor automatically generates a class diagram of the domain model. Instantly updated documentation!
* Possibility to deploy without EJBs, i.e. to run in Tomcat.


### Version 1.1.1

Version 1.1.1 was released November 26, 2007.

This release is a minor bug fix release.


## Version 1.0.x

### Version 1.0.0

Version 1.0.0 was released May 24, 2007.

The first final version of Sculptor has been released. The textual Domain Specific Language (DSL) and the design of theg enerated code are inspired by Domain-Driven Design. When using Sculptor you can focus on the business domain, instead of technical details. Sculptor uses openArchitectureWare to parse the DSL and generate high quality Java code and configuration. The generated code is based on well-known frameworks, such as Spring, Hibernate and Java EE.


### Version 1.0.1

Version 1.0.1 was released June 28, 2007.

This release is a minor bug fix release.


[1]: advanced-tutorial
[2]: maven-plugin
