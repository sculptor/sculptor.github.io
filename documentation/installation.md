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
      <li><a href="#eclipse">Eclipse</a></li>
      <li><a href="#sculptor-plugins">Sculptor Plugins</a></li>
      <li><a href="#eclipse-configuration">Eclipse Configuration</a></li>
      <li><a href="#maven-launcher">Maven Launcher</a></li>
    </ul>
  </div>
  <div class="span9">
    <div class="page-header">
      <h1>{{ page.title }}</h1>
    </div>
    <div markdown="1">

This instruction describes what you need to install and configure to be able to use Sculptor as normal developer, e.g. for getting started with the [Hello World Tutorial](tutorial/hello-world.html).


## JDK

Install a recent [JDK](http://www.oracle.com/technetwork/java/javase/downloads/) (Java 6 or newer).


## Maven

Maven is used for generating source code and building the system.

1. Install [Maven](http://maven.apache.org/download.html) (Version 3.0.4 or newer)
2. Define environment variable `JAVA_HOME` with path to JDK installation folder
3. Define environment variable `MAVEN_OPTS` with JVM arguments, e.g. for increase the heap size for better performance and avoiding out of memory

**Windows:**

~~~
set JAVA_HOME=c:\devtools\jdk1.6.0_03
set MAVEN_OPTS=-Xms128m -Xmx1024m
~~~

**Mac OS X:**

~~~
JAVA_HOME=/System/Library/Frameworks/JavaVM.framework/Versions/1.6.0/Home
MAVEN_OPTS="-Xms128m -Xmx1024m"
~~~


## Eclipse

Sculptor can be used with a text editor or any IDE, but if you are an Eclipse user it is recommended that you install Eclipse with the following plugins to be able to use Sculptors DSL editor with error highlight, code completion, and outline.

1. Install [Eclipse](http://www.eclipse.org/downloads/) (Juno or newer), Eclipse IDE for Java EE Developers
2. Directly in Eclipse (Help -> Install New Software) install from the Eclipse releases (e.g. Juno) update site the following plugins:
  * Modeling > MWE 2 runtime SDK 2.4.0 (or newer)
  * Modeling > Xtext SDK 2.4.1 (or newer)


## Sculptor Plugins

Sculptors Eclipse plugins are available from the following update sites:

* Releases: [https://raw.github.com/sculptor/repository/eclipse](https://raw.github.com/sculptor/repository/eclipse)
* Development Snapshots: [https://raw.github.com/sculptor/snapshot-repository/eclipse](https://raw.github.com/sculptor/snapshot-repository/eclipse)
 
Install "Sculptor DSL Feature".
If you are going to develop rich clients you should also install "Sculptor Rich Client Feature".


## Eclipse Configuration

In Eclipse the following configuration settings are required:

1. For the [Maven Launcher](#maven-launcher) add the 'String Substitution' variable `MAVEN_EXEC` with the full-qualified path to Mavens start script (Windows: `mvn.bat`, Unix: `mvn.sh`)  
![Maven Exec Variable](/images/documentation/installation/maven-exec-variable.png)
2. If you are using Mac OS X you should change the default file encoding in Eclipse to `ISO-8859-1` via "Preferences > General > Workspace"


## Maven Launcher

Maven can be executed from the command prompt, but when developing a better alternative is to run it inside Eclipse as an external tool. You can checkout an Eclipse project with some pre-configured launchers from Sculptors GitHub repository at [https://github.com/sculptor/sculptor/tree/master/devtools/maven-launcher/](https://github.com/sculptor/sculptor/tree/master/devtools/maven-launcher/). 

If this Eclipse project (with the Maven launchers) is **open** in your Eclipse workspace then the menu items for this launchers are available in the 'External Tools' menu.

![External Tools Menu](/images/documentation/installation/external-tools-menu.png)
  </div>
</div>
