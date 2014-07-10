---
layout: page
title: "Installation Guide"
description: ""
wide: true
hide_header: true
navbar_name: docs
---
<div class="row">
  <div class="sidebar span3">
    <ul id="sidenav" class="nav nav-list affix">
      <li class="active"><a href="#jdk">JDK</a></li>
      <li><a href="#maven">Maven</a></li>
      <li><a href="#graphviz">GraphViz</a></li>
      <li><a href="#eclipse">Eclipse</a></li>
      <li><a href="#sculptor-eclipse-plugins">&nbsp;&nbsp;&nbsp;Sculptor Eclipse Plugins</a></li>
      <li><a href="#eclipse-configuration">&nbsp;&nbsp;&nbsp;Eclipse Configuration</a></li>
      <li><a href="#eclipse-projects-and-maven-build-started-from-command-line">&nbsp;&nbsp;&nbsp;Eclipse and Maven build</a></li>
    </ul>
  </div>
  <div class="span9">
    <div class="page-header">
      <h1>{{ page.title }}</h1>
    </div>
    <div markdown="1">

This instruction describes what you need to install and configure to be able to use Sculptor as normal developer, e.g. for getting started with the [Hello World Tutorial](hello-world-tutorial).

**Table of Contents:**

* toc
{:toc}


## JDK

Sculptor is implemented in Java. So we need a Java development environment:

1. Install a recent [JDK](http://www.oracle.com/technetwork/java/javase/downloads/) (Java 6 or newer)
2. Define environment variable `JAVA_HOME` with path to JDK installation folder

**Windows:**

~~~ sh
set JAVA_HOME=c:\devtools\jdk1.7.0_51
~~~

**Mac OS X:**

~~~ sh
JAVA_HOME=/Library/Java/JavaVirtualMachines/jdk1.7.0_51.jdk/Contents/Home
export JAVA_HOME
~~~

Check from the command line that Java is configured properly, e.g.

~~~
$ java -version
java version "1.7.0_51"
Java(TM) SE Runtime Environment (build 1.7.0_51-b13)
Java HotSpot(TM) 64-Bit Server VM (build 24.51-b03, mixed mode)
~~~


## Maven

Maven is used for generating source code via [Sculptors Maven plugin](maven-plugin) and building the system. To create new Maven projects [Sculptors Maven archetypes](maven-archetypes) can be used (as shown in the [Archetype Tutorial](archetype-tutorial)).

1. Install [Maven](http://maven.apache.org/download.html) (Version 3.1.1 or newer)
2. Define environment variable `M2_HOME` with path to Maven installation folder
2. Define environment variable `MAVEN_OPTS` with JVM arguments, e.g. for increase the heap size for better performance and avoiding out of memory

   To prevent OutOfMemoryError when using Sculptors code generator it's neccesary to increase the [Oracle JVMs heap and permanent generation](https://blogs.oracle.com/jonthecollector/entry/presenting_the_permanent_generation) via `-Xmx1024m -XX:MaxPermSize=128m`.
   {: .alert }

**Windows:**

~~~ sh
set M2_HOME=C:\devtools\apache-maven-3.2.1\
set MAVEN_OPTS=-Xms128m -Xmx1024m -XX:MaxPermSize=128m
~~~

**Unix:**

~~~ sh
M2_HOME=C:\devtools\apache-maven-3.2.1\
MAVEN_OPTS="-Xms128m -Xmx1024m -XX:MaxPermSize=128m"
export M2_HOME MAVEN_OPTS
~~~

Check from the command line that Maven is configured properly, e.g.

~~~
$ mvn -v
Apache Maven 3.2.1 (ea8b2b07643dbb1b84b6d16e1f08391b666bc1e9; 2014-02-14T18:37:52+01:00)
Maven home: /Users/torsten/Develop/apache-maven-3.2.1
Java version: 1.7.0_51, vendor: Oracle Corporation
Java home: /Library/Java/JavaVirtualMachines/jdk1.7.0_51.jdk/Contents/Home/jre
Default locale: en_US, platform encoding: UTF-8
OS name: "mac os x", version: "10.9.3", arch: "x86_64", family: "mac"
~~~


## GraphViz

Sculptor generates several UML diagrams for the domain model. Theses diagrams are defined as textual [GraphViz](http://www.graphviz.org/) `.dot` files.
The goal `generate-images` of [Sculptors Maven plugin](maven-plugin) generates images (.png) from these `.dot` files. This plugin is included in the `pom.xml`
created by the Sculptor Maven archetypes, but you need to install [GraphViz](http://www.graphviz.org/) and have the `dot` executable in path.

If the GraphViz Maven `dot` executable is not available in path then the Maven build aborts with the error message `Executing 'dot' command failed`.
{: .alert }


## Eclipse

Sculptor can be used with any text editor or IDE. But if you are an [Eclipse](http://eclipse.org/) user it is recommended that you
[install Eclipse](http://eclipse.org/downloads/) with the following plugins to be able to use Sculptors DSL editor with error highlight, code completion, and outline.

1. Install [Eclipse](http://www.eclipse.org/downloads/) (Juno or newer), Eclipse IDE for Java EE Developers

   To prevent OutOfMemoryError when using the Sculptor editor you can add `-XX:MaxPermSize=128m` in `eclipse.ini`, which is located in the Eclipse installation directory.
   {: .alert }

2. Directly in Eclipse (Help -> Install New Software) install from the Eclipse releases (e.g. Kepler) update site the following plugins:
   * General Purpose Tools > [m2e - Maven Integration for Eclipse](http://www.eclipse.org/m2e/) 1.4.0 (or newer)
   * Modeling > Xtext SDK 2.5.0 (or newer)

     Xtext **version 2.5.0** (or newer) is required because Sculptors DSL text editor leverages public API introduced with this version!
     {: .alert .alert-error}


### Sculptor Eclipse Plugins

Sculptors Eclipse plugins are available from the update site [http://sculptorgenerator.org/updates/](http://sculptorgenerator.org/updates/).
 
Install "Sculptor DSL Editor".


### Eclipse Configuration

In Eclipse the following configuration settings are required:

1. To support [Xtend template expressions](http://www.eclipse.org/xtend/documentation.html#templates) (which are using the "guillemets" characters `«` and `»` for tag brackets) the default file encoding in Eclipse should be set to `UTF-8` or `ISO-8859-1` via "Preferences > General > Workspace"
2. Refresh in Eclipse is often time consuming. In the [m2e Eclipse plugin](http://www.eclipse.org/m2e/) Maven launch configurations (created via context menu "Run As > Maven build..." on the corresponding Maven project or Maven POM) you should enable the option "Refresh > Refresh resources upon completion > The project containing the selected resource". Don't use "The entire workspace".


### Eclipse projects and Maven build started from command line

After executing a Maven build from the command line the corresponding projects in Eclipse have to be refreshed manually! Thereafter you should not have any red crosses (problems) in Eclipse. Sometimes, validation errors in code generation files (.xtend) must be cleaned manually as well. This an be done with a "clean build" (using "Project > Clean...") of the corresponding Eclipse project.

  </div>
</div>
