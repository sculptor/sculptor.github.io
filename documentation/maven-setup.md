---
layout: page
title: Maven Setup
navbar_name: docs
---

### Preparing Maven "settings.xml" for Sculptor plugin

Prerequisite: [Maven 3](http://maven.apache.org) should be installed in you system. Check the command `mvn --version`. It should display your system and Maven properties.

Copy the below [Maven profiles](http://maven.apache.org/guides/introduction/introduction-to-profiles.html) tag into your [Maven "settings.xml"](http://maven.apache.org/settings.html). The file may be found (depending on your OS) in:

* Linux: `~/.m2`
* Windows: `C:\Documents and Settings\$USER$\.m2`
* Vista/Windows7: `C:\Users\$USER$\.m2`

If you prefer not to modify your user Maven settings, you can point to temporal settings by specifying the `-s` parameter:

~~~ sh
mvn -s path_to_settings.xml [other options] [<goal(s)>] [<phase(s)>]
~~~

If the `settings.xml` file does not exist, create the file and copy into it the following:

~~~ xml
<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0
                      http://maven.apache.org/xsd/settings-1.0.0.xsd">
  <profiles>
    <profile>
      <id>Sculptor</id>
      <activation>
        <activeByDefault>true</activeByDefault>
      </activation>
      <properties>
        <archetypeCatalog>https://raw.github.com/sculptor/repository/maven/archetype-catalog.xml</archetypeCatalog>
        <archetypeRepository>https://raw.github.com/sculptor/repository/maven</archetypeRepository>
      </properties>
      <repositories>
        <repository>
          <id>sculptor-releases</id>
          <name>Sculptor Maven repository for releases</name>
          <url>https://raw.github.com/sculptor/repository/maven</url>
          <snapshots>
            <enabled>false</enabled>
          </snapshots>
        </repository>
        <repository>
          <id>sculptor-snapshots</id>
          <name>Sculptor Maven repository for snapshots</name>
          <url>https://raw.github.com/sculptor/snapshot-repository/maven</url>
          <releases>
            <enabled>false</enabled>
          </releases>
        </repository>
      </repositories>
      <pluginRepositories>
        <pluginRepository>
          <id>sculptor-releases</id>
          <name>Sculptor Maven plugin repository for releases</name>
          <url>https://raw.github.com/sculptor/repository/maven</url>
          <snapshots>
            <enabled>false</enabled>
          </snapshots>
        </pluginRepository>
        <pluginRepository>
          <id>sculptor-snapshots</id>
          <name>Sculptor Maven plugin repository for snapshots</name>
          <url>https://raw.github.com/sculptor/snapshot-repository/maven</url>
          <releases>
            <enabled>false</enabled>
          </releases>
        </pluginRepository>
      </pluginRepositories>
    </profile>
  </profiles>
</settings>
~~~
