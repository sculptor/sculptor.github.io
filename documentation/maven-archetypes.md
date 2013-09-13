---
layout: page
title: "Sculptor Maven Archetypes"
description: ""
wide: true
hide_header: true
navbar_name: docs
---
<div class="row">
  <div class="sidebar span3">
    <ul id="sidenav" class="nav nav-list affix">
      <li class="active"><a href="#sculptor-maven-archetype-parent">sculptor-maven-archetype-parent</a></li>
      <li><a href="#sculptor-maven-archetype">sculptor-maven-archetype</a></li>
      <li><a href="#sculptor-maven-archetype-web">sculptor-maven-archetype-web</a></li>
      <li><a href="#sculptor-maven-archetype-ear">sculptor-maven-archetype-ear</a></li>
    </ul>
  </div>
  <div class="span9">
    <div class="page-header">
      <h1>{{ page.title }}</h1>
    </div>
    <div markdown="1">

Sculptor comes with a set of [Maven archetypes][1] which you be used to create skeleton projects. This is an easy way to get started using Sculptor as the archetypes generate a Maven `pom.xml` with the necessary Maven plugins (including [Sculptors Maven plugin][2]) and dependencies.

Check the [Archetype Tutorial][3] for an example on how to use Sculptors Maven archetypes.
{: .alert}


## sculptor-maven-archetype-parent

This archetypes creates a [Maven aggregator project][4] (POM module with suffix `-parent`) needed for a [Maven multi-module project][5]. This aggregator project references the [Maven project with the Sculptor business tier](#sculptor-maven-archetype) in its module list.

The following properties are supported by the archetype:

{:.table .table-striped .table-bordered .table-condensed}
|-------------------------+------------+------------------
| Property                | Default    | Purpose
|:------------------------|:-----------|:-----------------
| `-Dweb=[true|false]`    | `false`    | If set to `true` then an additional Maven project with a [web presentation tier](#sculptor-maven-archetype-web) (WAR module with suffix `-web`) is added to the module list in the POM.
| `-Dear=[true|false]`    | `false`    | If set to `true` then an additional Maven project which packages the Sculptor business tier into an [EAR (module with suffix `-ear`](#sculptor-maven-archetype-ear)) is added to the module list in the POM.

**Example:**

The following command (**one line** - indicated by the trailing `\`) creates the `helloworld-parent` project which references the EAR project `helloworld-ear` in its module list:

~~~
mvn archetype:generate -DinteractiveMode=false -DarchetypeGroupId=org.sculptor \
   -DarchetypeArtifactId=sculptor-maven-archetype-parent \
   -DarchetypeVersion=3.0.0-SNAPSHOT \
   -DarchetypeRepository=https://raw.github.com/sculptor/snapshot-repository/maven/ \
   -DgroupId=org.helloworld -DartifactId=helloworld-parent -Dpackage=org.helloworld \
   -Dversion=1.0-SNAPSHOT -Dear=true
~~~


## sculptor-maven-archetype

This archetypes creates a Maven project with the Sculptor business tier. By default this project is part of a Maven multi-module project. Therefore in the projects POM a [parent (aggregator) project](#sculptor-maven-archetype-parent) is referenced.

The following properties are supported by the archetype:

{:.table .table-striped .table-bordered .table-condensed}
|-----------------------------+------------+------------------
| Property                    | Default    | Purpose
|:----------------------------|:-----------|:-----------------
| `-Drest=[true|false]`       | `false`    | If set to `true` then the project is a WAR module which provides REST resources for its business services. Refer to the [REST Tutorial][6] for details.
| `-Dstandalone=[true|false]` | `false`    | If set to `true` then the project is **not** an part of a Maven multi-module project. So the generated POM has no reference to a parent POM. Therefore this projects POM contains all the settings from the parent POM as well.
| `-Dejb=[true|false]`        | `false`    | If set to `true` then the project is uses EJB3 instead of Spring. Therefore the project nature `pure-ejb3` is set in the generator configuration.
| `-Dear=[true|false]`        | `false`    | If set to `true` then the project is part of an EAR. Therefore the deployment mode `ear` is set in the generator configuration.
| `-Djboss=[true|false]`      | `false`    | If set to `true` then the project is a JAR or WAR module which is deployed to the JBoss application server.

**Example:**

The following command (**one line** - indicated by the trailing `\`) creates the `helloworld` project for a stand-alone WAR with a business tier which supports RESTful services:

~~~
mvn archetype:generate -DinteractiveMode=false -DarchetypeGroupId=org.sculptor \
   -DarchetypeArtifactId=sculptor-maven-archetype \
   -DarchetypeVersion=3.0.0-SNAPSHOT \
   -DarchetypeRepository=https://raw.github.com/sculptor/snapshot-repository/maven/ \
   -DgroupId=org.helloworld -DartifactId=helloworld -Dpackage=org.helloworld \
   -Dversion=1.0-SNAPSHOT -Drest=true -Dstandalone=true
~~~


## sculptor-maven-archetype-web

This archetypes creates a Maven project with the web presentation tier (WAR module with suffix `-web`). By default this project is part of a Maven multi-module project. Therefore in the projects POM [a parent (aggregator) project](#sculptor-maven-archetype-parent) is referenced.

The following properties are supported by the archetype:

{:.table .table-striped .table-bordered .table-condensed}
|-----------------------------+------------+------------------
| Property                    | Default    | Purpose
|:----------------------------|:-----------|:-----------------
| `-Dejb=[true|false]`        | `false`    | If set to `true` then the project is refering to a business tier module which uses EJB3 (Maven packaging type `EJB`) instead of Spring (Maven packaging type `JAR`).
| `-Dear=[true|false]`        | `false`    | If set to `true` then the project is part of an EAR. Therefore the deployment mode `ear` is set in the generator configuration.
| `-Drest=[true|false]`       | `false`    | If set to `true` then the project is a WAR module which provides REST resources for business services. Refer to the [REST Tutorial][6] for details.
| `-Djboss=[true|false]`      | `false`    | If set to `true` then the project is a WAR module which is deployed to the JBoss application server.

**Example:**

The following command (**one line** - indicated by the trailing `\`) creates the `helloworld-web` project for a WAR with a business tier (which uses EJB3 and supports RESTful services) and deployment as an EAR:

~~~
mvn archetype:generate -DinteractiveMode=false -DarchetypeGroupId=org.sculptor \
   -DarchetypeArtifactId=sculptor-maven-archetype-web \
   -DarchetypeVersion=3.0.0-SNAPSHOT \
   -DarchetypeRepository=https://raw.github.com/sculptor/snapshot-repository/maven/ \
   -DgroupId=org.helloworld -DartifactId=helloworld-web -Dpackage=org.helloworld \
   -Dversion=1.0-SNAPSHOT -Dejb=true -Dear=true -Drest=true
~~~


## sculptor-maven-archetype-ear

This archetypes creates a Maven project for an EAR (module with suffix `-ear`) which which packages the [Sculptor business tier](#sculptor-maven-archetype) and optionally a [web presentation tier](#sculptor-maven-archetype-web). This EAR project is expected to be part of a Maven multi-module project. Therefore in the projects POM [a parent (aggregator) project](#sculptor-maven-archetype-parent) is referenced.

The following properties are supported by the archetype:

{:.table .table-striped .table-bordered .table-condensed}
|-----------------------------+------------+------------------
| Property                    | Default    | Purpose
|:----------------------------|:-----------|:-----------------
| `-Dejb=[true|false]`        | `false`    | If set to `true` then the project is adding an business tier module which uses EJB3 (instead of Spring) as an EJB module instead as an JAR module.
| `-Dweb=[true|false]`        | `false`    | If set to `true` then the project is adding a [web presentation tier](#sculptor-maven-archetype-web) (WAR module with suffix `-web`) as a web module.

**Example:**

The following command (**one line** - indicated by the trailing `\`) creates the `helloworld-ear` project which packages an EJB3-based business tier and a web presentation tier:

~~~
mvn archetype:generate -DinteractiveMode=false -DarchetypeGroupId=org.sculptor \
   -DarchetypeArtifactId=sculptor-maven-archetype-ear \
   -DarchetypeVersion=3.0.0-SNAPSHOT \
   -DarchetypeRepository=https://raw.github.com/sculptor/snapshot-repository/maven/ \
   -DgroupId=org.helloworld -DartifactId=helloworld-ear -Dpackage=org.helloworld \
   -Dversion=1.0-SNAPSHOT -Dejb=true -Dweb=true
~~~


   [1]: http://maven.apache.org/guides/introduction/introduction-to-archetypes.html
   [2]: maven-plugin
   [3]: archetype-tutorial
   [4]: http://maven.apache.org/pom.html#Aggregation
   [5]: http://maven.apache.org/guides/mini/guide-multiple-modules.html
   [6]: rest-tutorial

  </div>
</div>
