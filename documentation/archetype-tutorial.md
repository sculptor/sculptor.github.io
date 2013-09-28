---
layout: page
title: "Archetype Tutorial"
description: ""
navbar_name: docs
---
{% include JB/setup %}

In this tutorial we will create a simple Java EE application from scratch using [Sculptors Maven archetypes][1]. We will start with a simple war, deployed in Jetty and using HSQLDB in-memory database. Later on it is illustrated how it can be converted to use MySQL and be deployed in JBoss.

Before you start you must follow the instructions in the **[Installation Guide][3]**.

**Table of Contents:**

* toc
{:toc}


## Part 1 - Setup Maven projects

In this first part we will setup the project structure for Maven and Eclipse.


### Create Maven projects

Our Java EE application consists of the following projects:

* `helloworld-parent` - Only a Maven project for building the other parts.
* `helloworld` - Business tier. Containing the services and domain objects.
* `helloworld-web` - Presentation tier. Web application with REST resources (as described in the [REST Tutorial][6]) for the services defined in the business tier.

We start with creating a script that calls the [Sculptor Maven archetypes][1] to generate the project structures and Maven build files. It also defines the environment variables (as described in the [installation guide][3]) and does an initial build. Of course you can execute these commands one by one from the command prompt, but the script is useful when doing this several times.

Copy one of the following scripts into the root of your Eclipse workspace.

 * Windows script `sculptor-archetypes.cmd`:

   ~~~
   set JAVA_HOME=c:\Program Files\Java\jdk1.7.0_25\
   set M2_HOME=C:\devtools\apache-maven-3.0.5\
   set MAVEN_OPTS=-Xms128m -Xmx1024m -XX:MaxPermSize=128m

   set path=%JAVA_HOME%\bin;%M2_HOME%\bin
   
   set PACKAGE=%1
   set SYS_NAME=%2

   set REPOSITORY=https://raw.github.com/sculptor/snapshot-repository/maven/
   set VERSION=3.0.0-SNAPSHOT
   
   mvn archetype:generate -DinteractiveMode=false -DarchetypeGroupId=org.sculptorgenerator -DarchetypeArtifactId=sculptor-maven-archetype-parent -DarchetypeVersion=%VERSION% -DarchetypeRepository=%REPOSITORY% -DgroupId=%PACKAGE% -DartifactId=%SYS_NAME%-parent -Dpackage=%PACKAGE% -Dversion=1.0-SNAPSHOT -Dweb=true
   
   mvn archetype:generate -DinteractiveMode=false -DarchetypeGroupId=org.sculptorgenerator -DarchetypeArtifactId=sculptor-maven-archetype -DarchetypeVersion=%VERSION% -DarchetypeRepository=%REPOSITORY% -DgroupId=%PACKAGE% -DartifactId=%SYS_NAME% -Dpackage=%PACKAGE% -Dversion=1.0-SNAPSHOT -Djboss=false
   
   mvn archetype:generate -DinteractiveMode=false -DarchetypeGroupId=org.sculptorgenerator -DarchetypeArtifactId=sculptor-maven-archetype-web -DarchetypeVersion=%VERSION% -DarchetypeRepository=%REPOSITORY% -DgroupId=%PACKAGE% -DartifactId=%SYS_NAME%-web -Dpackage=%PACKAGE% -Dversion=1.0-SNAPSHOT -Drest=true -Djboss=false
   
   pause
   
   call mvn install -f %SYS_NAME%-parent\pom.xml
   ~~~

 * Unix bash script `sculptor-archetypes.sh`:

   ~~~
   #!/bin/bash
   
   if [ -z $1 ] || [ -z $2 ]; then
      echo -e "Usage: $0 PACKAGEID ARTIFACTID\n\tPACKAGEID - name of Java package, for example org.helloworld"
      echo -e "\tARTIFACTID - project base name, for example helloworld"
      exit 1
   fi

   JAVA_HOME=/Library/Java/JavaVirtualMachines/jdk1.7.0_25.jdk/Contents/Home/
   M2_HOME=/opt/apache-maven-3.0.5
   MAVEN_OPTS="-Xms128m -Xmx1024m -XX:MaxPermSize=128m"

   path=$path:$JAVA_HOME/bin:$M2_HOME/bin

   export JAVA_HOME M2_HOME MAVEN_OPTS path 
   
   PACKAGE=$1
   SYS_NAME=$2
   
   REPOSITORY=https://raw.github.com/sculptor/snapshot-repository/maven/
   VERSION=3.0.0-SNAPSHOT

   mvn archetype:generate -DinteractiveMode=false -DarchetypeGroupId=org.sculptorgenerator -DarchetypeArtifactId=sculptor-maven-archetype-parent \
      -DarchetypeVersion=$VERSION -DarchetypeRepository=$REPOSITORY -DgroupId=$PACKAGE -DartifactId=$SYS_NAME-parent \
      -Dpackage=$PACKAGE -Dversion=1.0-SNAPSHOT -Dweb=true

   mvn archetype:generate -DinteractiveMode=false -DarchetypeGroupId=org.sculptorgenerator -DarchetypeArtifactId=sculptor-maven-archetype \
      -DarchetypeVersion=$VERSION -DarchetypeRepository=$REPOSITORY -DgroupId=$PACKAGE -DartifactId=$SYS_NAME \
      -Dpackage=$PACKAGE -Dversion=1.0-SNAPSHOT -Djboss=false

   mvn archetype:generate -DinteractiveMode=false -DarchetypeGroupId=org.sculptorgenerator -DarchetypeArtifactId=sculptor-maven-archetype-web \
      -DarchetypeVersion=$VERSION -DarchetypeRepository=$REPOSITORY -DgroupId=$PACKAGE -DartifactId=$SYS_NAME-web \
      -Dpackage=$PACKAGE -Dversion=1.0-SNAPSHOT -Drest=true -Djboss=false
   
   sleep 1
   
   mvn install -f $SYS_NAME-parent/pom.xml
   ~~~

   To make the script executable use run the command `chmod +x sculptor-archetypes.sh`.
   {: .alert}


You have to adjust the paths used in the variables of this scripts to your environment.
{: .alert .alert-success}


Run this script by defining the package name as the first parameter and the application id (artifact id of business tier) as the second parameter:

~~~
sculptor-archetypes org.helloworld helloworld
~~~


### Import Maven projects into Eclipse

Open Eclipse and import the newly created Maven projects with [m2e (Maven Integration for Eclipse)][4] via "File > Import... > Existing Maven Projects".

Then create the following m2e launch configurations via context menu "Run As > Maven build..." on the corresponding projects POM:

* For the project `helloworld-parent` use the Maven goals `clean install` to rebuild all projects.
* For the projects `helloworld` and `helloworld-web` use the Maven goal `generate-sources` to start Sculptors code generator. 

To refresh the Eclipse workspace after Sculptors code generator created new code enable the option "Refresh > Refresh resources upon completion > The project containing the selected resource" in the launch configurations.
{: .alert}


After executing a Maven build **from the commandline** the corresponding projects in Eclipse have to be refreshed manually (via "File > Refresh" or `F5`)

Sometimes, validation errors in code generation files (.xtend) must be cleaned manually as well. This can be done with a "clean build" (using "Project > Clean...") of the corresponding Eclipse project.
{: .alert}


## Part 2 - Generate Code

In this part we will write a Sculptor DSL file and generate code from it.


### Complete Business Tier

To allow us using the module with the business tier from within the presentation tier we have to create several model files (see [Advanced Tutorial](advanced-tutorial#divide-model-into-several-files)).
Open `model.btdesign` located in `src/main/resources` of the `helloworld` project and replace the content with the folowing model:

~~~
import "classpath:/model-planet.btdesign"

Application Universe {
    basePackage=org.helloworld
 
}
~~~

Here the actual model of the business tier is imported (as `ApplicationPart`) from a model file we're creating next.
Create the file `model-person.btdesign` in `src/main/resources` of the `helloworld` project and add something like this to the design file:

~~~
ApplicationPart Planet {
 
    Module planet {

        Service PlanetService {
            findById => PlanetRepository.findById;
            findAll => PlanetRepository.findAll;
            save => PlanetRepository.save;
            delete => PlanetRepository.delete;
        }
        
        Entity Planet {
            scaffold
            String name
            int diameter
            int population
            Repository PlanetRepository {
                findById;
                save;
                delete;
                findAll;
            }
        }
    }
}
~~~

Note the [scaffold feature](advanced-tutorial#scaffold) of `Planet`. It will result in automatically generated domain objects `PlanetRepository` and `PlanetService` with CRUD operations for the entity `Planet`.
{: .alert}


### Complete Presentation Tier

For the presentation we're using a simple CRUD gui generated from the REST resources defined for business services. For details refer to the [REST Tutorial][6].

In the presentation tier model we're referencing the model of the business tier which is already generated in the project `helloworld`. To skip generation of a module you define the module (`planet` in this example) in `sculptor-generator.properties` (as described in the [Advanced Tutorial](advanced-tutorial#cross-project-references)):

~~~
generate.module.planet=false
~~~

Now let's add the REST resources for the CRUD gui. Open `model.btdesign` located in `src/main/resources` of the `helloworld-web` project and add something like this to the design file:

~~~
import "classpath:/model-planet.btdesign"

Application Universe {
    basePackage=org.helloworld
    
    Module planetweb {
        Resource FrontResource {
            String front return="redirect:/rest/planet";
        }
        
        Resource PlanetResource {
            String createForm;
            create => PlanetService.save;
            show => PlanetService.findById;
            showAll => PlanetService.findAll;
            delete => PlanetService.delete;
        }
    }
}
~~~

Note the Resource operation `front` in the `FrontResource`. It represents the link named "Home" in the generated JSPs. In this case we're redirecting to the operation `showAll` in the 'PlanetResource'.
{: .alert}


### Launch Maven build

Start the Maven build from within Eclipse via "Run As > Maven Build" on the `helloworld-parent` project.

We skip the JUnit tests in this tutorial. See part 3 of the [Hello World Tutorial][5] for more information on fixing the failing JUnit tests.
{: .alert .alert-success}


### Run in Jetty

Start [Jetty][7] with `mvn jetty:run` in the `helloworld-web` project. Note that no installation is needed. Jetty is launched by Maven.

Open the URL [http://localhost:8888/](http://localhost:8888/) in your browser. Try the CRUD GUI and create some favourite Planets. The features of the generated web application are explained in the [REST Tutorial](rest-tutorial#crud-resource).

By default an in memory database, hsqldb, is bundled with the the application. Of course, when you restart the application or server all data added will be lost.
Jetty is an excellent development server, and you can stop here if you don't need to run in JBoss. You can also convert to JBoss later if you like.


## Part 3 - Deploy in JBoss

In this final part we're going to deploy our WAR in JBoss.


### Start JBoss

1. Download JBoss AS 7.2 from [here](http://www.jboss.org/jbossas/downloads) or [here](http://olex.openlogic.com/packages/jboss) and unpack the archive.

2. From within `JBOSS_HOME` [start JBoss in "standalone" mode](https://docs.jboss.org/author/display/AS72/Getting+Started+Guide#GettingStartedGuide-StartingJBossApplicationServer7), e.g. `./bin/standalone.sh`.

3. Test your JBoss AS installation by pointing your browser to [http://localhost:8080](http://localhost:8080) which brings you to the Welcome Screen


### Configure JBoss

Before we can deploy our WAR we have to prepare some resources within JBoss first:

1. JBoss AS 7 doesn't come with [HSQLDB](http://hsqldb.org/) anymore (instead JBoss uses [H2](http://www.h2database.com/) as in-memory database). So we have to install the HSQLDB JDBC driver first. For this we're using the [JBoss Command Line Client (CLI)](https://docs.jboss.org/author/display/AS72/Management+Clients) as described [here](http://www.mastertheboss.com/jboss-script/installing-a-jboss-as-7-module-using-the-cli). Locate the latest version of the HSQLDB JDBC driver JAR in you local Maven repository and fire up JBoss CLI:

   ~~~
   ./bin/jboss-cli.sh
   [disconnected /] connect
   [standalone@localhost:9999 /] module add --name=org.hsqldb --resources=/Users/torsten/Develop/maven-repos/org/hsqldb/hsqldb/2.2.9/hsqldb-2.2.9.jar --dependencies=javax.api,javax.transaction.api
   [standalone@localhost:9999 /] /subsystem=datasources/jdbc-driver=hsqldb:add(driver-module-name=org.hsqldb, driver-name=hsqldb, driver-class-name=org.hsqldb.jdbc.JDBCDriver)
   {"outcome" => "success"}
   ~~~

   In the above `module add` command you have to update the `resources` parameter with the full path to your local Maven repository!
   {: .alert .alert-info}

2. Next we need the datasource `UniversedDS` which is using HSQLDB. A deployable datasource configuration can be found in `src/generated/resources/dbschema/hellowrld-ds.xml`. But we're using JBoss CLI for this as well:

   ~~~
   [standalone@localhost:9999 /] /subsystem=datasources/data-source=UniverseDS:add(driver-name="hsqldb", connection-url="jdbc:hsqldb:mem:universe", jndi-name="java:/jdbc/UniverseDS", use-java-context=true, user-name="sa", password="sa")
   {"outcome" => "success"}
   [standalone@localhost:9999 /] /subsystem=datasources/data-source=UniverseDS:enable(persistent=true)
   {"outcome" => "success"}
   ~~~


### Adjust for JBoss

To deploy the WAR in JBoss the following adjustments have to be made:

 1. Since JBoss has some libraries in its default classpath you need to adjust several dependencies in `pom.xml` files (both business tier and presentation tier). Hibernate, slf4j and some more should not be bundled in WAR or EAR when deployed to JBoss.
To generate different POMs with the [Sculptor Maven archetypes][1] you have to set the property `-Djboss=` to `true` in the script `sculptor-archetypes` created in chapter "Part 1 - Setup Maven projects". Then delete **both** POM files and re-run the script `sculptor-archetypes`.

 2. To create code for JBoss instead of Jetty you have to adjust the following property in `sculptor-generator.properties` in `helloworld` **and** `helloworld-web` project.

    ~~~
    deployment.applicationServer=JBoss
    ~~~

 3. To get a proper context root for the deployed web app (the default one is the name of the WAR file) you have to create the JBoss deployment descriptor `jboss-web.xml` in the folder `src/main/webapp/WEB-INF/` of the `hellowworld-web` project with the following content:

    ~~~
    <?xml version="1.0" encoding="UTF-8"?>
	<jboss-web>
	    <context-root>/helloworld</context-root>
	</jboss-web>
	~~~

Rebuild with `mvn clean install` from the `helloworld-parent` project.


### Deploy to JBoss

Deploy the WAR file to JBoss via `mvn jboss-as:deploy` in the project `helloworld-web`.

Open [http://localhost:8080/helloworld](http://localhost:8080/helloworld) in your browser.


## Part 4 - Use MySQL Database

To use JBoss with [MySQL](http://www.mysql.com/) instead of HSQLDB the following adjustments are neccessary:

1. Adjust the following properties in `sculptor-generator.properties` in the project `helloworld` to let Sculptor generate a database creation sql script (DDL) for MySQL in `src/generated/resources/dbschema/`:

   ~~~
   db.product=mysql
   generate.ddl=true
   ~~~

2. Rebuild with `mvn clean install` from project `helloworld-parent`.

3. Create the database schema named `universe` in your [MySQL](http://www.mysql.com/) database and run the DDL SQL script in `helloworld/src/generated/resources/dbschema/` to create the database tables. You can use [MySQLWorkbench](http://dev.mysql.com/downloads/tools/workbench/) to do that or use the MySQL command line client:

   ~~~
   mysql -u root
   mysql> create database universe;
   mysql> connect universe;
   mysql> source helloworld/src/generated/resources/dbschema/Myapp_ddl.sql
   ~~~

4. JBoss AS 7 doesn't come with the MySQL JDBC driver. So we have to download the MySQL JDBC driver from [http://dev.mysql.com/downloads/connector/j/](http://dev.mysql.com/downloads/connector/j/) and install it first. For this we're using the [JBoss Command Line Client (CLI)](https://docs.jboss.org/author/display/AS72/Management+Clients) as described [here](http://www.mastertheboss.com/jboss-script/installing-a-jboss-as-7-module-using-the-cli):

   ~~~
   ./bin/jboss-cli.sh
   [disconnected /] connect
   [standalone@localhost:9999 /] module add --name=com.mysql --resources=/Users/torsten/Develop/maven-repos/mysql/mysql-connector-java/5.1.26/mysql-connector-java-5.1.26.jar --dependencies=javax.api,javax.transaction.api
   [standalone@localhost:9999 /] /subsystem=datasources/jdbc-driver=mysql:add(driver-module-name=com.mysql, driver-name=mysql, driver-class-name=com.mysql.jdbc.Driver)
   {"outcome" => "success"}
   ~~~

   In the above `module add` command you have to update the `resources` parameter with the full path to your MySQL JDBC driver!
   {: .alert .alert-info}

5. Next we have to replace the previously created datasource `UniversedDS` (which is using HSQLDB) by one using MySQL instead. A deployable datasource configuration can be found in `src/generated/resources/dbschema/hellowrld-ds.xml`. But we're using JBoss CLI for this as well:

   ~~~
   [standalone@localhost:9999 /] /subsystem=datasources/data-source=UniverseDS:remove
   {"outcome" => "success"}
   [standalone@localhost:9999 /] /subsystem=datasources/data-source=UniverseDS:add(driver-name="mysql", connection-url="jdbc:mysql://localhost/universe", jndi-name="java:/jdbc/UniverseDS", use-java-context=true, user-name="root")
   {"outcome" => "success"}
   [standalone@localhost:9999 /] /subsystem=datasources/data-source=UniverseDS:enable(persistent=true)
   {"outcome" => "success"}
   ~~~

6. Redeploy and test again.



   [1]: maven-archetypes
   [2]: http://maven.apache.org/guides/mini/guide-multiple-modules.html
   [3]: installation
   [4]: http://wiki.eclipse.org/M2E
   [5]: hello-world-tutorial
   [6]: rest-tutorial
   [7]: http://www.eclipse.org/jetty/
