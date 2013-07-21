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
      <li><a href="#sculptor-eclipse-plugins">Sculptor Eclipse Plugins</a></li>
      <li><a href="#eclipse-configuration">Eclipse Configuration</a></li>
      <li><a href="#maven-launcher">Maven Launcher</a></li>
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
set JAVA_HOME=c:\devtools\jdk1.7.0_25
~~~

**Mac OS X:**

~~~ sh
JAVA_HOME=/Library/Java/JavaVirtualMachines/jdk1.7.0_25.jdk/Contents/Home
~~~

Check from the command line that Java is configured properly, e.g.

~~~
$ java -version
java version "1.7.0_25"
Java(TM) SE Runtime Environment (build 1.7.0_25-b15)
Java HotSpot(TM) 64-Bit Server VM (build 23.25-b01, mixed mode)
~~~


## Maven

Maven is used for generating source code and building the system.

1. Install [Maven](http://maven.apache.org/download.html) (Version 3.0.5 or newer)
2. Define environment variable `MAVEN_OPTS` with JVM arguments, e.g. for increase the heap size for better performance and avoiding out of memory

   To prevent OutOfMemoryError when using Sculptors code generator it's neccesary to increase the [Oracle JVMs permanent generation](https://blogs.oracle.com/jonthecollector/entry/presenting_the_permanent_generation) via `-XX:MaxPermSize=128m`.
   {: .alert }

**Windows:**

~~~ sh
set MAVEN_OPTS=-Xms128m -Xmx1024m -XX:MaxPermSize=128m
~~~

**Mac OS X:**

~~~ sh
MAVEN_OPTS="-Xms128m -Xmx1024m -XX:MaxPermSize=128m"
~~~

Check from the command line that Maven is configured properly, e.g.

~~~
$ mvn -version
Apache Maven 3.0.5 (r01de14724cdef164cd33c7c8c2fe155faf9602da; 2013-02-19 14:51:28+0100)
Maven home: /Users/torsten/Develop/apache-maven-3.0.5
Java version: 1.7.0_25, vendor: Oracle Corporation
Java home: /Library/Java/JavaVirtualMachines/jdk1.7.0_25.jdk/Contents/Home/jre
Default locale: en_US, platform encoding: UTF-8
OS name: "mac os x", version: "10.8.3", arch: "x86_64", family: "mac"
~~~


## GraphViz

Sculptor generates several UML diagrams for the domain model. Theses diagrams are defined as textual [GraphViz](http://www.graphviz.org/) `.dot` files.
The goal `generate-images` of Sculptors Maven plugin generates images (.png) from these `.dot` files. This plugin is included in the `pom.xml`
created by the Sculptor Maven archetypes, but you need to install [GraphViz](http://www.graphviz.org/) and have the `dot` executable in path.

If the GraphViz Maven `dot` executable is not available in path then the Maven build aborts with the error message `Executing 'dot' command failed`.
{: .alert }


## Eclipse

Sculptor can be used with any text editor or IDE, but if you are an [Eclipse](http://eclipse.org/) user it is recommended that you
[install Eclipse](http://eclipse.org/downloads/) with the following plugins to be able to use Sculptors DSL editor with error highlight, code completion, and outline.

1. Install [Eclipse](http://www.eclipse.org/downloads/) (Juno or newer), Eclipse IDE for Java EE Developers
2. Directly in Eclipse (Help -> Install New Software) install from the Eclipse releases (e.g. Kepler) update site the following plugins:
   * Modeling > MWE 2 runtime SDK 2.4.0 (or newer)
   * Modeling > Xtext SDK 2.4.2 (or newer)

To prevent OutOfMemoryError when using the Sculptor editor you can add `-XX:MaxPermSize=128m` in `eclipse.ini`, which is located in the Eclipse installation directory.
{: .alert }


### Sculptor Eclipse Plugins

Sculptors Eclipse plugins are available from the following update sites:

* Releases: [https://raw.github.com/sculptor/repository/eclipse](https://raw.github.com/sculptor/repository/eclipse)
* Development Snapshots: [https://raw.github.com/sculptor/snapshot-repository/eclipse](https://raw.github.com/sculptor/snapshot-repository/eclipse)
 
Install "Sculptor DSL Feature".


### Eclipse Configuration

In Eclipse the following configuration settings are required:

1. For the [Maven Launcher](#maven-launcher) add the 'String Substitution' variable `MAVEN_EXEC` with the full-qualified path to Mavens start script (Windows: `mvn.bat`, Unix: `mvn.sh`)  
![Maven Exec Variable](/images/documentation/installation/maven-exec-variable.png)
2. If you are using Mac OS X you should change the default file encoding in Eclipse to `ISO-8859-1` via "Preferences > General > Workspace"


### Maven Launcher

Maven can be executed from the command prompt, but when developing a better alternative is to run it inside Eclipse as an external tool. You can checkout an Eclipse project with some pre-configured launchers from Sculptors GitHub repository at [https://github.com/sculptor/sculptor/tree/master/devtools/maven-launcher/](https://github.com/sculptor/sculptor/tree/master/devtools/maven-launcher/). 

If this Eclipse project (with the Maven launchers) is **open** in your Eclipse workspace then the menu items for this launchers are available in the 'External Tools' menu.

![External Tools Menu](/images/documentation/installation/external-tools-menu.png)

  </div>
</div>
