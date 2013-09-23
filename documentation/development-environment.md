---
layout: page
title: "Development Environment"
description: ""
navbar_name: docs
---

This document describes the development environment for the Sculptor project:

* toc
{:toc}


## Installation

To develop Sculptor you need local installations of the following tools:

* [Java JDK](http://www.oracle.com/technetwork/java/javase/downloads/) (1.6 or newer)
* [Maven](http://maven.apache.org/download.html) (3.0.5 or newer)
* [Eclipse](http://eclipse.org/downloads/) (4.2 or newer) with [Xtext](http://www.eclipse.org/Xtext/download.html) (2.4.2 or newer)
* (optional) [GraphViz](http://www.graphviz.org/) (2.2.8 or newer)

The installation and configuration of these tools is described in [Sculptors installation guide](http://sculptorgenerator.org/documentation/installation).


### Maven Repository Manager

To improve development experience it's recommended to install a Maven repository manager like [Sonatype Nexus](http://www.sonatype.org/nexus/). Instead letting the Sculptors Maven build access the external Maven repositories directly the repository manager is used as a proxy. This proxy greatly reduces the network access because all Maven artifacts are only retrieved once and then managed by the local repository manager.


## Configuration

After the successfully installation (as described in [Sculptors installation guide](http://sculptorgenerator.org/documentation/installation)) some additional configurations are necessary.


### Eclipse text file encoding

Sculptors code generator templates are implemented with [Xtext template expressions](http://www.eclipse.org/xtend/documentation.html#templates). These templates are using special tag brackets (guillemets), for which the characters `«` (Unicode `00AB`) and `»` (Unicode `00BB`) are used. The downside with the guillemets used in the templates is that you will have to have a consistent encoding (here `ISO-8859-1`).

So it's necessary to set the default encoding in Eclipse to **ISO-8859-1** via "Preferences > General > Workspace > Text file encoding > ISO-8859-1".
{: .alert .alert-success }

As mentioned [here](http://www.eclipse.org/Xtext/documentation.html#encoding) this encoding is used in the development of Xtext as well. 


### Maven settings.xml

The [Maven "settings.xml"](http://maven.apache.org/settings.html) needs some customizations. This file may be found (depending on your OS) in:

* *nix and Mac: `~/.m2`
* Windows: `C:\Documents and Settings\$USER$\.m2`
* Vista/Windows7: `C:\Users\$USER$\.m2`

In this file we have to make the following modifications:

#### Maven repository manager
{: #repo-manager-profile}

To use a local Maven repository manager (here Nexus) for redirecting all requests to the Maven Central repository a [Maven profile](http://maven.apache.org/guides/introduction/introduction-to-profiles.html) and corresponding [`<mirror/>` settings](http://maven.apache.org/guides/mini/guide-mirror-settings.html) are necessary, e.g.

~~~ xml
<profiles>
  <profile>
    <id>nexus</id>
    <mirrors>
      <!--This sends all request for the dummy URL http://central to the local Nexus instance -->
      <mirror>
        <id>nexus</id>
        <mirrorOf>central</mirrorOf>
        <url>http://localhost:8081/nexus/content/groups/public/</url>
      </mirror>
    </mirrors>

    <!-- Enable redirection of Maven Central to local Nexus via mirror with dummy URL http://central -->
    <repositories>
      <repository>
        <id>central</id>
        <url>http://central</url>
        <releases><enabled>true</enabled></releases>
        <snapshots><enabled>true</enabled></snapshots>
      </repository>
    </repositories>
   <pluginRepositories>
      <pluginRepository>
        <id>central</id>
        <url>http://central</url>
        <releases><enabled>true</enabled></releases>
        <snapshots><enabled>true</enabled></snapshots>
      </pluginRepository>
    </pluginRepositories>
  </profile>
</profiles>

<activeProfiles>
  <!--make the profile active all the time -->
  <activeProfile>nexus</activeProfile>
</activeProfiles>
~~~

Usage of the local repository manager can be (temporarily) turned-off by disabling the coresponding Maven profile via the commandline option `-P!nexus`.
{: .alert .alert-info }


#### Eclipse p2 repository mirror
{: #p2-mirror-profile}

To use the [local Eclipse p2 repository mirror](#p2-mirror) (created by the Sculptor build via Eclipse Tycho) for requests to the external Eclipse p2 repositories an additional Maven profile with corresponding [`<mirror/>` settings](http://maven.apache.org/guides/mini/guide-mirror-settings.html) are necessary, e.g.

~~~ xml
<profiles>
  <profile>
    <id>p2-mirror</id>
    <mirrors>
      <mirror>
        <!-- This sends requests to p2 repositories to local mirror -->
        <id>mirror</id>
        <mirrorOf>p2.eclipse,p2.eclipse.xtext,p2.xtext-utils</mirrorOf>
        <url>"location of your Sculptor repository clone"/devtools/eclipse-mirror/.p2-mirror/</url>
        <layout>p2</layout>
        <mirrorOfLayouts>p2</mirrorOfLayouts>
      </mirror>
    </mirrors>
  </profile>
</profiles>

<activeProfiles>
  <!--make the profile active all the time -->
  <activeProfile>p2-mirror</activeProfile>
</activeProfiles>
~~~

Usage of the local p2 repository mirror can be (temporarily) turned-off by disabling the coresponding Maven profile via the commandline option `-P!p2-mirror`.
{: .alert .alert-info }


#### Deployment to GitHub
{: #github-credentials}

For deploying to GitHub the GitHub user credentials have to be defined in the `<servers/>` section, e.g.

~~~ xml
<servers>
  <server>
    <id>github</id>
    <username>johndoo</username>
    <password>xyz</password>
  </server>
</servers>
~~~


## Source Code

Use Eclipses Git support [EGit](http://www.eclipse.org/egit/) or any other [Git client](http://git-scm.com/downloads) to clone Sculptors GitHub repository [https://github.com/sculptor/sculptor](https://github.com/sculptor/sculptor), e.g.

~~~
$ git clone git://github.com/sculptor/sculptor.git
$ cd sculptor
~~~


### Maven modules

The following is a brief overview of the main Maven modules of Sculptors source code: 

* `devtools` Folder with projects used for local development, e.g. `eclipse-mirror` with the local Eclipse p2 repository mirror.

* `releng` Folder with projects used by release engineering, e.g. `sculptor-parent` with the parent POM used by the other modules or `sculptor-distribution` with profiles used for building the Sculptor distribution.

* `sculptor-eclipse` The aggregator project holding the Eclipse projects with the Eclipse p2 mirror, the meta model, the DSL model with its UI and unit tests, the feature and the p2 mirror.

* `sculptor-generator` The aggregator project holding the implementation of the code generator.

* `sculptor-maven` The aggregator project holding the Maven plugin, the Maven archetypes and the Maven repository.


### Eclipse projects

Import all Maven projects in Eclipse (via [Eclipse M2E](http://wiki.eclipse.org/M2E)) with "File > Import... > Existing Maven Projects". For improved development experience organize the projects in appropriate [Eclipse working sets](http://help.eclipse.org/kepler/index.jsp?topic=%2Forg.eclipse.platform.doc.user%2Fconcepts%2Fcworkset.htm).

After executing a Maven build from the commandline the corresponding projects in Eclipse have to be refreshed manually! Thereafter you should not have any red crosses (problems) in Eclipse. Sometimes, validation errors in code generation files (.xtend) must be cleaned manually as well. This an be done with a "clean build" (using "Project > Clean...") of the corresponding Eclipse project.
{: .alert .alert-success}


### Eclipse Maven Launcher

Import the Maven project `devtools/maven-launcher` into Eclipses workspace as described in the [Installation Guide](installation#maven-launcher) (if you haven't already done that).

When this Eclipse project is open in the workspace you can run the Maven build from inside Eclipse. The corresponding menu items for this are available in Eclipses external tools menu.


## Build

The following chapters are describing the different aspects of Sculptors Maven-based build process.


### Creation of local Eclipse p2 repository mirror
{: #p2-mirror}

To improve the development experience the needed external Eclipse p2 repositories are "mirrored" via Eclipse Tycho. This mirror is located in the folder "devtools/eclipse-mirror/.p2-mirror/". The mirroring process is activated by using the Maven profile `mirror`:

~~~
cd devtools/sculptor-distribution
mvn initialize -Pmirror
~~~

The *initial* mirroring process takes hours!!!
{: .alert .alert-error}

Make sure that the [Maven profile used for the local Eclipse p2 repository mirror](#p2-mirror-profile) is added to the Maven "settings.xml".
{: .alert}


### Installation of custom Eclipse IDE

By using Eclipse Tycho it's possible to create a customized Eclipse installation. This Eclipse installation is located in the folder "devtools/eclipse-ide/target/products/org.sculptor.ide/<platform>". The installation process is activated by using the Maven profile `ide`:

~~~
cd releng/sculptor-distribution
mvn verify -Pide
~~~


### Build the whole project

Without specifying any Maven profile the whole project is built:

~~~
cd releng/sculptor-distribution
mvn clean install
~~~


### Build parts of the project

By specifying one or more from the following Maven profiles certain parts of the project are built:

* `eclipse` Sculptor Eclipse plugin
* `maven` Sculptor Maven plugin and archetypes
* `example` Sculptor examples
* `all` The whole Sculptor project (same as specifying no profile)

E.g. to built the Maven stuff and the examples only then the following commands are used:

~~~
cd releng/sculptor-distribution
mvn clean install -Pmaven,example
~~~


### Deployment to GitHub

To deploy the Maven repository (with Sculptors Maven plugin and the coresponding archtypes) and the Eclipse p2 repository (with the Sculptor Eclipse plugin) to GitHub the Maven deploy plugin is used. To activate the transfer to the GitHub repository the Maven profile `deploy` has to be specified:

~~~
cd releng/sculptor-distribution
mvn deploy -Pdeploy
~~~

The corresponding approach is described [here](http://stackoverflow.com/questions/14013644/hosting-a-maven-repository-on-github/).
{: .alert}

Make sure that your Maven "settings.xml" contains the correct [GitHub user credentials](#github-credentials).
{: .alert .alert-error}


### Stand-alone Code Generator

To build the stand-alone Generator JAR the Maven profile `shade` is used:

~~~
cd releng/sculptor-distribution
mvn install -Pshade
cd ../../sculptor-generator/sculptor-core
java -jar target/sculptor-core-3.0.0-SNAPSHOT.jar -model src/test/resources/model-test.btdesign
~~~

The generator isn't useful right now - it reads the model, validates it and prints "org.eclipse.emf.mwe2.runtime.workflow.Workflow - Done.".
