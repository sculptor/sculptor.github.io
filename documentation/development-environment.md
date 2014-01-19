---
layout: page
title: "Development Environment"
description: ""
navbar_name: docs
---

This document describes the development environment and the development / release process for the Sculptor project:

* toc
{:toc}


## Installation

To develop Sculptor you need local installations of the following tools:

* [git](http://git-scm.com/downloads) (1.8 or newer)
  * git workflow extension [git-flow](https://github.com/nvie/gitflow)
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

The repository IDs listed in the <mirrorOf> tag have to match the repository IDs used in the Maven POM of the Maven (aggregator) project  `sculptor-eclipse`.
{: .alert .alert-error }

Usage of the local p2 repository mirror can be (temporarily) turned-off by disabling the coresponding Maven profile via the commandline option `-P!p2-mirror`.
{: .alert .alert-info }


#### Deployment to GitHub
{: #github-credentials}

For deploying the Eclipse plugin to GitHub the GitHub user credentials have to be defined in the `<servers/>` section, e.g.

~~~ xml
<servers>
  <server>
    <id>github</id>
    <username>your-username</username>
    <password>your-password</password>
  </server>
</servers>
~~~

The server ID is referenced by the [GitHub site-maven-plugin](https://github.com/github/maven-plugins) used in the `eclipse-repository` project.
{: .alert .alert-info }


#### Deployment to Sonatype OSS Repository Hosting Service
{: #sonatype-credentials}

For deploying the Maven plugin to Sonatypes OSS Repository Hosting (OSSRH) service the Sonatype user credentials have to be defined in the `<servers/>` section, e.g.

~~~ xml
<servers>
  <server>
    <id>sonatype-nexus-snapshots</id>
    <username>your-username</username>
    <password>your-password</password>
  </server>
  <server>
    <id>sonatype-nexus-staging</id>
    <username>your-username</username>
    <password>your-password</password>
  </server>
</servers>
~~~

The server IDs are referenced by the distribution repositories defined the Sonatype `oss-parent` POM referenced in the `sculptor-parent` project.
{: .alert .alert-info }


#### Passphrase for code signing via GPG Maven plugin
{: #gpg-passphrase}

For signing the Maven artifacts via the [maven-gpg-plugin](http://maven.apache.org/plugins/maven-gpg-plugin/) during the release build a Maven profile with the GPG passphrase has to be defined in the `<profiles/>` section, e.g.

~~~ xml
<profile>
  <id>release-gpg-passphrase</id>
    <activation>
      <property>
        <name>performRelease</name>
        <value>true</value>
      </property>
    </activation>
  <properties>
    <gpg.passphrase>your-passphrase</gpg.passphrase>
  </properties>
</profile>
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

* `sculptor-eclipse` The aggregator project holding the Eclipse projects with the Eclipse p2 mirror, the meta model, the DSL model with its UI and unit tests, the feature and the p2 mirror.

* `sculptor-generator` The aggregator project holding the implementation of the code generator.

* `sculptor-framework` The aggregator project holding the implementation of the Sculptor runtime library.

* `sculptor-maven` The aggregator project holding the Maven plugin, the Maven archetypes and the Maven repository.

* `sculptor-examples` The aggregato project holding a few examples which are using Scupltor, eg. the ones used in the tutorials.


### Eclipse projects

Import all Maven projects in Eclipse (via [Eclipse M2E](http://wiki.eclipse.org/M2E)) with "File > Import... > Existing Maven Projects". For improved development experience organize the projects in appropriate [Eclipse working sets](http://help.eclipse.org/kepler/index.jsp?topic=%2Forg.eclipse.platform.doc.user%2Fconcepts%2Fcworkset.htm).

After executing a Maven build from the commandline the corresponding projects in Eclipse have to be refreshed manually! Thereafter you should not have any red crosses (problems) in Eclipse. Sometimes, validation errors in code generation files (.xtend) must be cleaned manually as well. This an be done with a "clean build" (using "Project > Clean...") of the corresponding Eclipse project.
{: .alert .alert-success}


## Build

The following chapters are describing the different aspects of Sculptors Maven-based build process.


### Creation of local Eclipse p2 repository mirror
{: #p2-mirror}

To improve the development experience the needed external Eclipse p2 repositories are "mirrored" via Eclipse Tycho. The mirroring process is activated by using the Maven profile `mirror`:

~~~
mvn initialize -Pmirror
~~~

The *initial* mirroring process takes hours!!! After the build is finished then the mirror can be found in the folder "devtools/eclipse-mirror/.p2-mirror/". 
{: .alert}

Make sure that the [Maven profile used for the local Eclipse p2 repository mirror](#p2-mirror-profile) is added to the Maven "settings.xml".
{: .alert .alert-error}


### Installation of custom Eclipse IDE

By using Eclipse Tycho it's possible to create a customized Eclipse installation. The installation process is activated by using the Maven profile `ide`:

~~~
mvn verify -Pide
~~~

After the build is finished then the Eclipse installation can be found in the folder "devtools/eclipse-ide/target/products/org.sculptor.ide/<platform>".
{: .alert}


### Build the whole project

Without specifying any Maven profile the whole project is built:

~~~
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
mvn clean install -Pmaven,example
~~~


## Development process

The project uses the [gitflow workflow](https://www.atlassian.com/git/workflows#!workflow-gitflow). So development of small features is done in the **branch "develop"**. Larger features are implemented within a feature branch (created via `git flow feature start some-new-feature`).

After finishing the new feature the feature branch is merged with "develop" via `git flow feature finish some-new-feature`.

Non-committers are not merging the feature branch with "develop" (within their forked repository) but pushing the feature branch to their GitHub repository and [opening a pull request](https://help.github.com/articles/creating-a-pull-request). After the pull request gets accepted the feature is merged in the "develop" branch of the Sculptor repository. From here the merged "develop" branch (with the feature) can be pulled in the forked repository.
{: .alert .alert-info }

The **branch "master"** is used for released code and the corresponding release tags only!
{: .alert .alert-error }


## Release process

Sculptors release artifacts are hosted at different locations:

* Sculptors Eclipse plugins p2 repository is hosted on GitHub in the folder "updates" of [Sculptors website repository](https://github.com/sculptor/sculptor.github.io).
* Sculptors Maven plugin and archeytpes are hosten on Maven Central.

To release Sculptors Eclipse plugin and the Maven plugins (including the archetypes) we don't use the (official) [Maven release plugin](http://maven.apache.org/maven-release/maven-release-plugin) but Atlassians [Maven JGitFlow Plugin](https://bitbucket.org/atlassian/maven-jgitflow-plugin) is used. It is based on and is a replacement for the maven-release-plugin enabling support for git-flow style releases via Maven.

Due to JGitFlows missing support for the Eclipse config files `MANIFEST.MF` and `feature.xml` (it only updates the version numbers in Maven POMs) we're using the [Tycho versions plugin](http://eclipse.org/tycho/sitedocs/tycho-release/tycho-versions-plugin/plugin-info.html) for this.

So Sculptors release process is as follows:

* create a release branch and updates pom(s) with the release version via `mvn jgitflow:release-start -P!all -DreleaseVersion=<RELEASE_VERSION>`
* update the Eclipse config files with the release version via `mvn tycho-versions:set-version -P!all -DnewVersion=<RELEASE_VERSION>` and commit the modified files
* runs a Maven build (deploy or install), merges the release branch, updates pom(s) with the development version via `mvn jgitflow:release-finish -DdevelopmentVersion=<DEVLOPMENT_VERSION>`
* update the Eclipse config files with the development version via `mvn tycho-versions:set-version -P!all -DnewVersion=<DEVLOPMENT_VERSION>` and commit the modified files

To automate this process the script file [`release.sh`](https://github.com/sculptor/sculptor/blob/develop/release.sh) is available. This script requires as arguments the release version and the next development version (without the suffix "-SNAPSHOT"), e.g. `./release 3.0.0 3.0.1`.
{: .alert .alert-success}

The following chapters are describing the deployment of the Eclipse plugin and the Maven plugin during the release build (executed via `mvn jgitflow:release-finish`).


### Deployment of Eclipse p2 repository to GitHub

To deploy the Eclipse p2 repository (with the Sculptor Eclipse plugin) to GitHub the [GitHub site-maven-plugin](https://github.com/github/maven-plugins) is used in the `eclipse-repository` project. The corresponding approach is described [here](http://stackoverflow.com/questions/14013644/hosting-a-maven-repository-on-github/).

Make sure that your Maven "settings.xml" contains the correct [GitHub user credentials](#github-credentials).
{: .alert .alert-error}


### Deployment of Maven artifacts to Sonatypes OSS

To deploy the Maven artifacts to Maven Central (via Sonatypes OSS Repository Hosting as described in [OSSRH-5507](https://issues.sonatype.org/browse/OSSRH-5507)) the [maven-deploy-plugin](http://maven.apache.org/plugins/maven-deploy-plugin/) is used.

Make sure that your Maven "settings.xml" contains the correct [Sonatype user credentials](#sonatype-credentials).
{: .alert .alert-error}

