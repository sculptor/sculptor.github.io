---
layout: page
title: "Developers Guide"
description: ""
navbar_name: docs
---
{% include JB/setup %}

Sculptor is not an one-size-fits-all product. Even though it is a good start for many systems, sooner or later customization is always needed.
This guide will give you an understanding of the internal design of Sculptor and explains how to do changes for some typical scenarios.
Some things can easily be changed with properties or AOP, while other things requires more effort, since you need to setup the [development environment][1].
Some changes are staightforward and some requires more in depth understanding of Eclipse [Xtext](http://www.eclipse.org/xtext/) and [Xtend](http://www.eclipse.org/xtend/).

**Table of Contents:**

* toc
{:toc}


## Sculptor Internal Design

![Sculptors internal design](/images/documentation/developers-guide/sculptor-internal-design.png) \\
<small>_Figure 1. Internal Design of Sculptor_</small>

1. The developer is using the DSL Editor plugin to edit the application specific `model.btdesign`, i.e. the source for the concrete model that is the input to the code generation process. Constraints of the DSL is validated while editing.

2. When generating code the application specific `Workflow.mwe2` is executed. It doesn't contain much.

3. It invokes the `Sculptorworkflow.mwe2` which defines the flow of the code generation process.

4. It starts with parsing the `model.btdesign` file using the XText parser. Constraints of the DSL are checked.

5. The DSL model is transformed into a model of the type defined by the `sculptormetamodel.ecore` meta model.

6. Constraint validation is performed.

7. The model is transformed again. Now it is actually modified to add some default values.

8. Constraint validation again.

9. Now the actual generation of Java code and configuration files begin. It is done with code generation templates written in [Xtends template expressions](http://www.eclipse.org/xtend/documentation.html#templates). The templates extract values from the model and uses [Xtend extension methods](http://www.eclipse.org/xtend/documentation.html#extensionMethods) and Java helper classes.

10. Properties of technical nature, which don't belong in the DSL or meta model, are used by the templates and the helpers.


## Performance Tuning of Generator

By using one model file per module it is possible for Sculptor to do a partial generate of the changed modules and the ones depending on the changed modules. The file must be named the same as the module (e.g. `media.btdesign`, `person.btdesign`) or prefixed with `model_` or `model-` (model-person.btdesign). This partial generation can shorten the generation time for large projects. [`sculptor-maven-plugin`][3] will detect which model files has changed since previous generator run when using `mvn -o generate-sources`. Full generate will be done when using `-Dsculptor.generator.force=true` or `mvn clean generate-sources`

The Maven `-o` (offline) option can reduce Maven execution time a lot, if you know that you have everything locally.
{: .alert}

I normally use `mvn -o generate-sources` when doing small changes, which is most of the time, and then `mvn clean generate-sources` or `mvn clean install`when I have done major changes, or want to ensure that everything is working.

A major disadvantage of using `mvn clean` is that Eclipse will often not understand the changes and a separate (and annoying) Eclipse refresh and project clean is needed.

Refresh in Eclipse is often time consuming. In the m2e launch configuration used for executing Maven you should enable the option "Refresh > Refresh resources upon completion > The project containing the selected resource". Don't use "The entire workspace".
{: .alert}

[`sculptor-maven-plugin`][3] runs the generator within the Maven process and therefore it can be necessary to define JVM parameters for large projects via the environment variable `MAVEN_OPTS`:

* Windows: `set MAVEN_OPTS="-Xms256m -Xmx1g -XX:MaxPermSize=256m"`
* Unix: `export MAVEN_OPTS=-Xms256m -Xmx1g -XX:MaxPermSize=256m`


## Generator Properties

There are a many things that can be easily customized with properties. The default properties are defined in `default-sculptor-generator.properties` in [`sculptor-generator-core`](https://github.com/sculptor/sculptor/tree/master/sculptor-generator/sculptor-generator-core). You can override these properties by defining them in `sculptor-generator.properties` and `sculptor-gui-generator.properties`. You only have to define the ones that you need to change.

Sculptor will look for properties in the following order:

  1. `System.properties` (only intended for temporary tests)
  2. `sculptor-gui-generator.properties`
  3. `sculptor-generator.properties`
  4. `default-sculptor-generator.properties`


### Project Nature

The main capabilities and responsibilities of the project is defined by the `project.nature` property. It is typically used to assign default values of more fine grained properties. E.g. the generated artifact of a presentation tier project is different from a business tier project.

The default project nature is `business-tier`. For a web project you specify:

~~~
project.nature=presentation-tier
~~~

A project may have several natures and the value of the property can be a comma separated list. E.g. to generate business and presentation tier in the same project you can specify:

~~~
project.nature=presentation-tier, business-tier
~~~


### Framework Classes

You can replace the runtime framework with your own implementations by defining the class names in `sculptor-generator.properties`. The naming convention is that the property key is the last part, starting with `framework`, of the default Sculptor framework class.
E.g. to use your own base class for all domain objects:

~~~
framework.domain.AbstractDomainObject=org.myown.framework.MyAbstractDomainObject
~~~


### Generic Access Objects

Sculptor provides generic access objects for operations such as `findById`, `findAll`, `save`, `delete`, etc. The implementation of these operations are located in the runtime framework. It is easy to replace those with your own implementation.

To replace all of them you define the following properties:

~~~
framework.accessimpl.package=org.mypkg.accessimpl
framework.accessimpl.prefix=
framework.accessapi.package=org.mypkg.accessapi
~~~

To replace a single implementation you define the class name like this:

~~~
framework.accessimpl.SaveAccessImpl=org.mypkg.accessimpl.SaveAccessImpl
~~~

If you need to replace the interface also:

~~~
framework.accessapi.SaveAccess=org.mypkg.accessapi.SaveAccess
framework.accessimpl.SaveAccessImpl=org.mypkg.accessimpl.SaveAccessImpl
~~~

When using Hibernate as JPA provider (which is the default) then the Hibernate specific implementations are used. For example `findByCriteria` is Hibernate specific. In case you want to use Hibernate as JPA provider, but stick to pure JPA implementations then you should add the following properties:

~~~
framework.accessimpl.prefix=Jpa
framework.accessimpl.package=org.sculptor.framework.accessimpl.jpa
~~~


### Database Product

MySQL, Oracle, PostgreSQL and HSQLDB-InMemory are supported out-of-the-box. You can select database with

~~~
db.product=mysql
#db.product=oracle
#db.product=postgresql
#db.product=hsqldb-inmemory
~~~

Contribution of support for Apache Derby has been posted in the [Forum](http://www.mail-archive.com/fornax-developer@lists.sourceforge.net/msg00437.html)

The database properties are only used for generating the DDL and some Hibernate settings.
You can also change the default type mapping:

~~~
db.mysql.type.Boolean=CHAR(1)
db.mysql.type.boolean=CHAR(1)
db.mysql.type.Integer=INTEGER
db.mysql.type.int=INTEGER
db.mysql.type.Long=BIGINT
db.mysql.type.long=BIGINT
db.mysql.type.IDTYPE=BIGINT
db.mysql.type.Date=DATE
db.mysql.type.java.util.Date=TIMESTAMP
db.mysql.type.DateTime=DATETIME
db.mysql.type.Timestamp=TIMESTAMP
db.mysql.type.BigDecimal=DOUBLE
db.mysql.type.BigInteger=BIGINT
db.mysql.type.Double=DOUBLE
db.mysql.type.double=DOUBLE
db.mysql.type.Float=FLOAT
db.mysql.type.float=FLOAT
db.mysql.type.String=VARCHAR
db.mysql.length.String=100
db.mysql.type.UUID=VARCHAR
db.mysql.length.UUID=36
db.mysql.type.Enum=VARCHAR
db.mysql.length.Enum=40
db.mysql.type.Clob=CLOB
db.mysql.type.Blob=BLOB
db.mysql.type.discriminatorType.STRING=VARCHAR
db.mysql.length.discriminatorType.STRING=31
db.mysql.type.discriminatorType.CHAR=CHAR(1)
db.mysql.type.discriminatorType.INTEGER=INTEGER
~~~

For parent child (one-to-many) relations the database can do cascaded delete of children when deleting parent. This is the default setting. It is possible to disable that and let Hibernate do the delete by setting the following property to `false`.

~~~
db.mysql.onDeleteCascade=false
~~~


#### DDL script

There are two ways to generate database creation SQL script (DDL). Select between:

1. Enable the `hibernate3-maven-plugin hbm2ddl` in `pom.xml`. The archetype generates settings for it, but it is commented out.
The default settings is to generate the ddl script every time in the build phase `process-classes`. You can change that by
removing the `<phase>process-classes</phase>` in the `execution` section of the plugin definition in `pom.xml`. In that case
you need to run the generation when needed with `mvn hibernate3::hbm2ddl`.

2. Sculptor will generate DDL script if you add the following property in `sculptor-generator.properties`:

   ~~~
   generate.ddl=true
   ~~~

The generated DDL script contains drop statements. In case you don't wan't to generate those you can define:

~~~
generate.ddl.drop=false
~~~

Individual classes may be omitted from the generated DDL. This can be useful to keep parts of the model that are works in progress from the DDL. To omit a class from the generated DDL, mark it with a 'skipddl' hint:

~~~
Entity Roles {
	hint="skipddl"
	scaffold

	String roleName length="50" key;
}
~~~

By default, some of the system-generated attributes, such as `version`, `createdBy`, `createdDate`, are ordered at the end of each generated table in the DDL. This can be controlled via the 'systemAttributesToPutLast' property:

~~~
# Attributes defined by system that should appear last in attribute listings, such as in DDL
systemAttributesToPutLast=version,createdBy,createdDate,updatedBy,updatedDate,lastUpdated,lastUpdatedBy
~~~


### Cascade

Default cascade values can be defined for different types of associations. References between different modules have no default cascade value. `all` is default for references within the same module. You might want to change that to `persist,merge` if you don't want delete to be propagated.

~~~
# default cascade values for aggregate references
cascade.aggregate=all
# bidirectional one-to-many for aggregate reference
cascade.aggregate.oneToMany=all-delete-orphan
# default cascade values for references within same module
# bidirectional many-to-many
cascade.manyToMany=all
# bidirectional one-to-many
cascade.oneToMany=all
# unidirectional to-many
cascade.toMany=all
# unidirectional to-one
cascade.toOne=all
~~~


### Types

Mapping from simple types used in the DSL to generated types are defined with these properties:

~~~
javaType.Date=java.util.Date
javaType.DateTime=java.util.Date
javaType.Timestamp=java.util.Date
javaType.Map=java.util.Map
javaType.List=java.util.List
javaType.Set=java.util.Set
javaType.Collection=java.util.Collection
~~~

`java.lang` types, such as `String` and `Integer` are not defined, since they have the same name in the DSL and the Java code.
{: .alert}

You can add your own types also, e.g.

~~~
javaType.AccountNumber=org.foobar.type.AccountNumber
javaType.Currency=String
javaType.Amount=BigDecimal
~~~

When you use your own classes, such as `AccountNumber`, you need to define the corresponding Hibernate user type.

~~~
hibernateType.AccountNumber=org.foobar.type.PersistentAccountNumber
~~~

If you define the above types you can use them directly in the DSL.

~~~
Entity Account {
  AccountNumber accountNumber
  - @Money balance
}

BasicType Money {
  Amount amount
  Currency currency
}
~~~

Another example, `ShortString`, which combines the database and java type properties.

~~~
db.mysql.type.ShortString=VARCHAR
db.mysql.length.ShortString=10
javaType.ShortString=String
~~~

The type of the automatically generated id attribute is defined as

~~~
id.type=Long
~~~


### Joda Date and Time API
{: #joda}

It is possible to use [Joda Time](http://joda-time.sourceforge.net/) instead of the Java date and time classes.

~~~
datetime.library=joda
~~~


### Package Names
{: #packageNames}

You can define the names of the generated packages.

~~~
package.serviceInterface=serviceapi
package.serviceImplementation=serviceimpl
package.serviceProxy=serviceproxy
package.serviceStub=richclient.servicestub
package.consumer=consumer
package.rest=rest
package.xmlmapper=consumer
package.domain=domain
package.builder=domain
package.commandEvent=domain
package.domainEvent=domain
package.repositoryInterface=domain
package.exception=exception
package.repositoryImplementation=repositoryimpl
package.accessInterface=accessimpl
package.accessImplementation=accessimpl
package.mapper=mapper
package.web=web
package.richClient=richclient
package.common=common
package.dto=serviceapi
~~~

You can also use subpackages.

~~~
package.domain=domain
package.repositoryInterface=domain.repository
package.exception=domain.exception
~~~


### Default Base Class

It is possible to define your own class to be use as extended base class for different types.

~~~
repository.extends=org.foo.SuperRepository
service.extends=org.foo.SuperService
entity.extends=org.foo.SuperEntity
valueObject.extends=org.foo.SuperValue
dataTransferObject.extends=org.foo.SuperDTO
basicType.extends=org.foo.SuperType
domainEvent.extends=org.foo.SuperEvent
commandEvent.extends=org.foo.SuperCommand
~~~

Note that extends can also be defined per domain object in `model.btdesign`:

~~~
Entity Movie extends @Media {
}

DataTransferObject SearchResponse extends org.foo.ResponseBase {
}
~~~


### Database Naming

There are a few configuration options for different naming strategies of database elements.

Convert camel-case to underscore in database names:

~~~
db.useUnderscoreNaming=true
~~~

Foreign key column names ending with `_ID`:

~~~
db.useIdSuffixInForeigKey=true
~~~

ID column prefixed with table name:

~~~
db.useTablePrefixedIdColumn=true
~~~


### File Header

You can define a header to be generated in java files. The java comments are added automatically, but you need to include the newline character.

~~~
javaHeader=Copyright line 1 \n\
 line 2 \n\
 line 3
~~~


### toString

[Apache commons-langs *ReflectionToStringBuilder*](http://commons.apache.org/proper/commons-lang/javadocs/api-2.6/org/apache/commons/lang/builder/ReflectionToStringBuilder.html) is used for `toString()` in DomainObjects. You can define style with property:

~~~
toStringStyle=SHORT_PREFIX_STYLE
#toStringStyle=MULTI_LINE_STYLE
#toStringStyle=NO_FIELD_NAMES_STYLE
#toStringStyle=SIMPLE_STYLE
~~~

JavaDoc of [ToStringStyle](http://commons.apache.org/proper/commons-lang/javadocs/api-2.6/org/apache/commons/lang/builder/ToStringStyle.html) describes the different formats.

You can also override this for individual DomainObject with `hint`:

~~~
ValueObject Foo {
    hint="toStringStyle=MULTI_LINE_STYLE"
    String aaa
    String bbb
}
~~~


### equals and hashCode

For composite natural keys an inner key class is generated. If you prefer to skip this inner class you can define this property:

~~~
generate.domainObject.compositeKeyClass=false
~~~

One drawback with not using key class is that `findByKeys` is not supported for composite keys.
{: .alert}


### Removing the generation of the ServiceContext
{: #serviceContext}

For some reason it might be required to switch off the generation of the `ServiceContext` - for example if you want to create a client without the created/updated functionality.
In this case it can be done easily with setting a property value in the `sculptor-generator.properties` file:

~~~
# specifies if ServiceContext is to be generated
generate.serviceContext=false
~~~

This turns off the generation of the `ServiceContext`. Please note that in this case the [auditable feature][4] does not work or must be implemented manually.

You can skip generation of auditable with:

~~~
generate.auditable=false
~~~


### Null instead of NotFoundException

By default, some repository operations such as `findById` and `findByKey` throws `NotFoundException` if the query doesn't find any matching result. You can use the following generator property to return `null` instead of throwing exception:

~~~
generate.NotFoundException=false
~~~


### XML configuration for Spring

If you prefer Spring bean definitions in xml files instead of annotations you can use this property:

~~~
generate.spring.annotation=false
~~~


### XML mapping for Hibernate

If you prefer hibernate mapping in xml files (hbm) instead of JPA annotations you can use this property:

~~~
generate.jpa.annotation=false
~~~


### Cache Provider

Cache provider settings are generated based on the property `cache.provider`.

~~~
cache.provider=EhCache
#cache.provider=JbossTreeCache
#cache.provider=DeployedTreeCache
#cache.provider=TreeCache
#cache.provider=Infinispan
~~~

There are some built in cache providers.

  1. EhCache - (default) settings are defined in `ehcache.xml` file, skeleton is generated
  2. JbossTreeCache - settings are defined as a JBoss mbean `EJB3EntityTreeCache`
  3. DeployedTreeCache - settings are defined as a JBoss mbean
  4. TreeCache - settings are defined in `treecache.xml` file
  5. Infinispan - (default cache provider for JBoss) no settings required because it's pre-configured in JBoss AS 7

The choice also affects the cache `usage` attribute in the Hibernate mapping.

EhCache is the default cache provider, since it doesn't require any additional configuration. JBoss `Infinispan` is recommended for (clustered) applications that require a transactional cache. Refer to JBoss documentation for information about how to configure [Infinispan](https://www.jboss.org/infinispan).
{: .alert .alert-success}


### Validation

In case you don't want validation annotations you can omit them with:

~~~
generate.validation.annotation=false
~~~

By default validation annotations are not generated for DataTransferObjects, but that can be enabled with property:

~~~
generate.validation.annotation.dataTransferObject=true
~~~

You can add your own validations or replace the implementation of the built in validators:

~~~
# built-in validation annotation
validation.annotation.AssertFalse=javax.validation.constraints.AssertFalse
validation.annotation.AssertTrue=javax.validation.constraints.AssertTrue
validation.annotation.Digits=javax.validation.constraints.Digits
validation.annotation.Future=javax.validation.constraints.Future
validation.annotation.Max=javax.validation.constraints.Max
validation.annotation.Min=javax.validation.constraints.Min
validation.annotation.DecimalMax=javax.validation.constraints.DecimalMax
validation.annotation.DecimalMin=javax.validation.constraints.DecimalMin
validation.annotation.NotNull=javax.validation.constraints.NotNull
validation.annotation.Null=javax.validation.constraints.Null
validation.annotation.Past=javax.validation.constraints.Past
validation.annotation.Pattern=javax.validation.constraints.Pattern
validation.annotation.Size=javax.validation.constraints.Size
validation.annotation.Valid=javax.validation.Valid

# built-in validation annotation (provider hibernate)
validation.annotation.CreditCardNumber=org.hibernate.validator.constraints.CreditCardNumber
validation.annotation.Email=org.hibernate.validator.constraints.Email
validation.annotation.NotEmpty=org.hibernate.validator.constraints.NotEmpty
validation.annotation.NotBlank=org.hibernate.validator.constraints.NotBlank
validation.annotation.Range=org.hibernate.validator.constraints.Range
validation.annotation.Length=org.hibernate.validator.constraints.Length
validation.annotation.ScriptAlias=org.hibernate.validator.constraints.ScriptAlias
validation.annotation.URL=org.hibernate.validator.constraints.URL

# validation annotation for Joda API
validation.annotation.JodaFuture=org.sculptor.framework.validation.constraints.Future
validation.annotation.JodaPast=org.sculptor.framework.validation.constraints.Past
~~~


### JPA annotation settings

If you prefer JPA annotations on getters instead of fields:

~~~
generate.jpa.annotation.onField=false
~~~

To generate annotations for database definition of column type:

~~~
generate.jpa.annotation.columnDefinition=true
~~~


### JPA provider settings

If you prefer to use another JPA provider instead of Hibernate you can choose between EclipseLink, DataNucleus and OpenJPA.


#### EclipseLink

To use EclipseLink add

~~~
jpa.provider=eclipselink
~~~

Replace the hibernate properties and dependencies from `pom.xml` with

~~~ xml
<eclipselink.version>2.3.0</eclipselink.version>
<eclipselink-api.version>2.0.3</eclipselink-api.version>

<dependency>
  <groupId>org.eclipse.persistence</groupId>
  <artifactId>javax.persistence</artifactId>
  <version>${eclipselink-api.version}</version>
</dependency>
<dependency>
  <groupId>org.eclipse.persistence</groupId>
  <artifactId>eclipselink</artifactId>
  <version>${eclipselink.version}</version>
</dependency>
~~~
    
add the plugin to enhance the domain classes and the repository to resolve the dependencies

~~~ xml
<plugin>
  <artifactId>maven-antrun-plugin</artifactId>
  <executions>
    <execution>
      <phase>process-classes</phase>
      <configuration>
        <tasks>
          <java classname="org.eclipse.persistence.tools.weaving.jpa.StaticWeave"
                classpathref="maven.compile.classpath" fork="true">
            <arg line="-loglevel FINE target/classes target/classes"/>
          </java>
        </tasks>
      </configuration>
      <goals>
        <goal>run</goal>
      </goals>
    </execution>
  </executions>
</plugin>

<repository>
  <id>EclipseLink Repo</id>
  <url>http://www.eclipse.org/downloads/download.php?r=1&amp;nf=1&amp;file=/rt/eclipselink/maven.repo</url>
</repository>
~~~


#### DataNucleus

If you want to use DataNucleus add

~~~
jpa.provider=datanucleus
~~~

Replace the related hibernate properties and dependencies from `pom.xml with

~~~ xml
<datanucleus.version>3.0.2</datanucleus.version>
<datanucleus.plugin.version>3.0.0-release</datanucleus.plugin.version>
<datanucleus.enhancer.version>3.0.0-release</datanucleus.enhancer.version>
<datanucleus.jpa.version>3.0.2</datanucleus.jpa.version>
<datanucleus.jodatime.version>3.0.0-release</datanucleus.jodatime.version>

<dependency>
	<groupId>org.datanucleus</groupId>
	<artifactId>datanucleus-core</artifactId>
	<version>${datanucleus.version}</version>
</dependency>
<dependency>
	<groupId>org.datanucleus</groupId>
	<artifactId>datanucleus-enhancer</artifactId>
	<version>${datanucleus.enhancer.version}</version>
</dependency>
<dependency>
	<groupId>org.datanucleus</groupId>
	<artifactId>datanucleus-rdbms</artifactId>
	<version>${datanucleus.version}</version>
</dependency>
<dependency>
	<groupId>org.datanucleus</groupId>
	<artifactId>datanucleus-api-jpa</artifactId>
	<version>${datanucleus.jpa.version}</version>
</dependency>
~~~    

add the plugin to enhance the domain classes and the repository to resolve the dependencies

~~~ xml
<plugin>
	<groupId>org.datanucleus</groupId>
	<artifactId>maven-datanucleus-plugin</artifactId>
	<version>${datanucleus.plugin.version}</version>
	<configuration>
		<metadataIncludes>**/domain/*.class</metadataIncludes>
		<metadataExcludes>**/domain/*Propert*.class,**/domain/*Repository.class</metadataExcludes>
		<api>JPA</api>
		<verbose>false</verbose>
		<ddlFile>${basedir}/src/test/generated/resources/dbunit/ddl.sql</ddlFile>
		<completeDdl>true</completeDdl>
	</configuration>
	<dependencies>
		<dependency>
			<groupId>org.datanucleus</groupId>
			<artifactId>datanucleus-core</artifactId>
			<version>${datanucleus.version}</version>
		</dependency>
		<dependency>
			<groupId>org.datanucleus</groupId>
			<artifactId>datanucleus-enhancer</artifactId>
			<version>${datanucleus.enhancer.version}</version>
		</dependency>
		<dependency>
			<groupId>org.datanucleus</groupId>
			<artifactId>datanucleus-rdbms</artifactId>
			<version>${datanucleus.version}</version>
		</dependency>
		<dependency>
			<groupId>org.datanucleus</groupId>
			<artifactId>datanucleus-api-jpa</artifactId>
			<version>${datanucleus.jpa.version}</version>
		</dependency>
		<dependency>
			<groupId>hsqldb</groupId>
			<artifactId>hsqldb</artifactId>
			<version>1.8.0.10</version>
		</dependency>
	</dependencies>
	<executions>
		<execution>
			<id>enhance</id>
			<phase>process-classes</phase>
			<goals>
				<goal>enhance</goal>
			</goals>
		</execution>
		<execution>
			<id>test-schema-create</id>
			<phase>process-test-classes</phase>
			<goals>
				<goal>schema-create</goal>
			</goals>
			<configuration>
				<props>${basedir}/src/test/generated/resources/datanucleus-test.properties</props>
			</configuration>
		</execution>
	</executions>
</plugin>

<repository>
    <id>DataNucleus</id>
    <name>DataNucleus Repository</name>
    <url>http://www.datanucleus.org/downloads/maven2/</url>
</repository>
<pluginRepository>
    <id>DataNucleus</id>
    <url>http://www.datanucleus.org/downloads/maven2/</url>
    </pluginRepository>
</pluginRepositories>
~~~


#### OpenJPA

To use OpenJPA add

~~~
jpa.provider=openjpa
~~~

Replace the hibernate properties and dependencies from `pom.xml` with

~~~ xml
<properties>
	<openjpa.version>2.2.0-SNAPSHOT</openjpa.version>
</properties>

<dependency>
	<groupId>org.apache.openjpa</groupId>
	<artifactId>openjpa</artifactId>
	<version>${openjpa.version}</version>
</dependency>
~~~

add the plugin to enhance the domain classes and the repository to resolve the dependencies

~~~ xml
<plugin>
	<groupId>org.codehaus.mojo</groupId>
	<artifactId>openjpa-maven-plugin</artifactId>
	<version>1.2</version>
	<configuration>
             <includes>**/domain/*.class</includes>
             <excludes>**/domain/*Propert*.class,**/domain/*Repository.class</excludes>
	</configuration>
	<executions>
		<execution>
			<id>enhancer</id>
			<phase>process-classes</phase>
			<goals>
				<goal>enhance</goal>
			</goals>
		</execution>
	</executions>
	<dependencies>
		<dependency>
			<groupId>org.apache.openjpa</groupId>
			<artifactId>openjpa</artifactId>
			<version>${openjpa.version}</version>
		</dependency>
	</dependencies>
</plugin>

<repository>
    <id>apache.snapshots</id>
    <name>Apache Snapshot Repository</name>
    <url>http://repository.apache.org/snapshots</url>
    <releases>
        <enabled>false</enabled>
    </releases>
</repository>
~~~


### Business Component without Persistence

You can create a business component that is not using JPA or MongoDB by defining generator property:

~~~
jpa.provider=none
~~~

Remove persistence related dependencies in `pom.xml`:

* joda-time-hibernate
* hsqldb
* dbunit
* commons-pool
* ehcache-core
* hibernate-validator
* hibernate-entitymanager
* hibernate-annotations
* hibernate-core
* persistence-api

If you are using the `jpa.provider=none` then you must add the classifier `without-jpa` to the `sculptor-framework-test` dependency:

~~~ xml
<dependency>
	<groupId>org.sculptorgenerator</groupId>
	<artifactId>sculptor-framework-test</artifactId>
	<version>${sculptor.version}</version>
	<scope>test</scope>
	<classifier>without-jpa</classifier>
</dependency>
~~~

The reason for using another jar file is that one of the test beans `JpaTestBean` should not be deployed.
{: .alert}


### Settings for Generated Diagrams
{: #diagram}

By default Sculptor generates a dot file (handled by [Graphviz](http://www.graphviz.org)) for documentation purposes. This might not be needed for all projects. Please set the following property to drive the generation of this feature.

~~~
# specifies if umlgraph for Graphviz is to be generated
generate.umlgraph=false
~~~

You can change colours for the generated diagrams.

~~~
umlgraph.bgcolor.Module=CCCCFF
umlgraph.bgcolor.Service=CCFF99
umlgraph.bgcolor.Consumer=CCFFFF
umlgraph.bgcolor.BasicType=D0D0D0
umlgraph.bgcolor.Enum=E0E0E0
umlgraph.bgcolor.Entity=FFCC33
umlgraph.bgcolor.ValueObject=FFFF99
umlgraph.bgcolor.DataTransferObject=FFCC99
umlgraph.bgcolor.DomainEvent=FFCC99
umlgraph.bgcolor.CommandEvent=FFFF99

# Highlight core domain with other colors
#umlgraph.fontcolor.core.Module=mediumblue
umlgraph.fontcolor.core.Service=mediumblue
umlgraph.fontcolor.core.Consumer=mediumblue
umlgraph.fontcolor.core.BasicType=mediumblue
umlgraph.fontcolor.core.Enum=mediumblue
umlgraph.fontcolor.core.Entity=mediumblue
umlgraph.bgcolor.core.Entity=FFCC66
umlgraph.fontcolor.core.ValueObject=mediumblue
umlgraph.bgcolor.core.ValueObject=FFFF99
umlgraph.fontcolor.core.DataTransferObject=mediumblue
umlgraph.fontcolor.core.DomainEvent=mediumblue
umlgraph.fontcolor.core.CommandEvent=mediumblue
umlgraph.labeldistance=2.0
umlgraph.labelangle=-30
~~~

It is also possible to define color for individual elements using hint in model `hint="umlgraph.bgcolor=D0D0D0"`. Hide elements using `hint="umlgraph=hide"`.


### Deployment in Tomcat

By default [Jetty](http://www.mortbay.org/jetty/) is used to run the application. Instead of Jetty you can use [Tomcat](http://tomcat.apache.org/).
Deployment in Tomcat requires that you do the following.

1. Define the following property in `sculptor-generator.properties` in the business tier project:

   ~~~
deployment.applicationServer=Tomcat
   ~~~

2. The Tomcat definition of the datasource is located in `META-INF/context.xml` in the WAR. This file is generated once, but you might need to adjust the settings.

3. Copy the jdbc driver jar to tomcat lib directory.

4. Rebuild with `mvn clean install` in the parent directory.

5. The WAR located in the target directory of the web project can be deployed in Tomcat.


### Deployment as EAR or WAR

Sculptor supports deployment as EAR or WAR. WAR is default when creating new projects with the archetypes. The design differences are:

* Services are exposed as EJBs when deployed as EAR, POJOs when deployed as WAR.
* Transaction management is done with JTA by the application server when deployed as EAR, by Spring when deployed as WAR.
* Consumers are not supported when deployed as WAR.

The [Archetype Tutorial](archetype-tutorial) describes the steps how to convert WAR to EAR deployment (e.g. by setting `deployment.type=ear`). It also describes how to deploy in JBoss (e.g. by setting `deployment.applicationServer=JBoss`).


### JAXB

JAXB annotations are by default only generated for DTOs. It can be turned on for other domain object types with these properties:

~~~
generate.xml.bind.annotation.dataTransferObject=true
generate.xml.bind.annotation.valueObject=false
generate.xml.bind.annotation.entity=false
generate.xml.bind.annotation.basicType=false
generate.xml.bind.annotation.domainEvent=false
generate.xml.bind.annotation.commandEvent=false
~~~


### XStream

You should add the following property to your `sculptor-generator.properties` to avoid fully qualified class names in the XML of the REST representations. `@XStreamAlias` will be generated.

~~~
generate.xstream.annotation=true
~~~


### Scaffold

[Scaffold][2] operations are defined as a comma separated list.

~~~
scaffold.operations=findById,findAll,save,delete
~~~


### Visibility of setters for not changeable attributes and references

By default the visibility is private for setter methods of not changeable attributes. Some tools might need visible setter methods and you can change the visibility with the following property. When you make these setters visible they will check that the value is not changed, once it has been set.

~~~
notChangeablePropertySetter.visibility=private
#notChangeablePropertySetter.visibility=protected
~~~

The visibility of setters for not changeable references are public. These setters will also check that the reference value is not changed, once it has been set. The reason for having the public setters is that sometimes the referred object is not available at construction time. Then it is possible to pass in null in the constructor and then afterwards assign the reference. If you would like to hide these setters you can use the following property.

~~~
notChangeableReferenceSetter.visibility=private
~~~


### Spring dispatcher servlet mapping

When generating the web client the default Spring dispatcher servlet mapping is `/spring/*`. To change it is a matter of changing a property:

~~~
gui.springServletMapping=foobarbaz
~~~

Now the mapping will be `/foobarbaz/*`.


### Spring Remoting Type

[Spring remoting](http://static.springsource.org/spring/docs/3.0.x/spring-framework-reference/html/remoting.html) with RMI is used by default. The type of Spring remoting can be selected with properties in `sculptor-generator.properties`:

~~~
# The type of remoting can be selected with
spring.remoting.type=rmi
#spring.remoting.type=hessian
#spring.remoting.type=burlap
#spring.remoting.type=httpInvoker
~~~

Hessian, Burlap and HttpInvoker requires some additional configuration in `web.xml`, as described in [Spring remoting documentation](http://static.springsource.org/spring/docs/3.0.x/spring-framework-reference/html/remoting.html).


## Spring Configuration

You can change the settings in `spring.properties`. It is runtime configuration of Spring.

Spring makes it possible to override bean definitions. The last definition wins. It is `applicationContext.xml` that is loaded first and it imports other configuration files. `more.xml` is imported last, which means that you can override generated bean definitions by specifying them in `more.xml`. This is also true for beans defined with annotations (`@Repository`, `@Service`, `@Component`). When running JUnit tests the corresponding file is `more-test.xml`.

This approach is not recommended, since it might be confusing to have multiple bean definitions. It is also easy to forget to change when something is renamed and that can have dangerous side effects. For testing or temporary modifications it might be an easy solution, but for permanent customization it is better to use one of the other alternatives presented in this article.


## Change Generation Templates

You can customize the code generation templates using the [Sculptor extension mechanism](advanced-tutorial.html#overrides-and-extension-mechanism).

See [Sculptor extension mechanism](advanced-tutorial.html#overrides-and-extension-mechanism) for details and steps on how to override Templates (or transformations) in your project, or to extend Sculptor via cartridges.

You find the default templates in [https://github.com/sculptor/sculptor/tree/master/sculptor-generator/sculptor-generator-core/src/main/java/org/sculptor/generator/template](https://github.com/sculptor/sculptor/tree/master/sculptor-generator/sculptor-generator-core/src/main/java/org/sculptor/generator/template).
Everything starts in [RootTmpl](https://github.com/sculptor/sculptor/blob/master/sculptor-generator/sculptor-generator-core/src/main/java/org/sculptor/generator/template/RootTmpl.xtend), which you also can intercept to add more templates or exclude some of the existing templates.

Another alternative is to setup the [development environment][1] and change the original templates and build a new version of [`sculptor-generator-core`](https://github.com/sculptor/sculptor/tree/master/sculptor-generator/sculptor-generator-core).

### hint
{: #hint}

A very useful extension mechanism is available via the `hint` keyword in the model. It is possible to use a hint on almost any element in the model. It may contain any key/values, which can be used in `SpecialCases` to customize the code generation. For example, if you need to use a special database sequence for the ids of some entities. In the model you can use the hint:

~~~
Entity Person {
    hint="idSequence=SEQ2"
~~~

In `SpecialCases.xpt`:

~~~
«AROUND templates::domain::DomainObjectAttributeAnnotation::idAnnotations FOR Attribute»
    «IF getDomainObject().hasHint("idSequence")»
        @javax.persistence.Id
        @javax.persistence.GeneratedValue(strategy=javax.persistence.GenerationType.SEQUENCE,
            generator="«getDomainObject().getHint('idSequence')»")
        @javax.persistence.Column(name="«getDatabaseName()»")
    «ELSE»
    	«targetDef.proceed()»
    «ENDIF»
«ENDAROUND»
~~~

Several hints can be defined, separated with comma, and the hint may have a value or not.

~~~
String someAttribute hint="aaa=A,bbb,ccc"
~~~


## Reference Application

When you customize Sculptor it is important that you have a representative reference application. It is used to verify the changes to the code generator. It must be big enough to span most of the design concepts you use, but still small enough to be supple.

When you do refactoring of the code generator it is recommended that you first do manual refactoring of the reference application. Keep that code base as a baseline. When you have changed the code generation the generated code should correspond to the manual baseline. Use a diff tool to compare the results.

The library example can act as a reference application if you haven't developed one of your own.
The source code is available in Sculptor GitHub repository: [https://github.com/sculptor/sculptor/tree/master/sculptor-examples/library-example](https://github.com/sculptor/sculptor/tree/master/sculptor-examples/library-example)

Run the JUnit tests to verify that it works before you start changing.

It is convenient to use Eclipse project dependencies from the reference application to [`sculptor-generator-core`](https://github.com/sculptor/sculptor/tree/master/sculptor-generator/sculptor-generator-core). Then you can do changes in the templates and try them directly, without having to build with Maven. You can run the code generation directly inside Eclipse. In your reference application project, right click on `Workflow.mwe2` and select 'Run as MWE workflow'.


## Constraint Validation

[Check Language](http://www.openarchitectureware.org/pub/documentation/4.3.1/html/contents/core_reference.html#Check_language) is used for validation of model constraints.


### DSL Constraints

DSL constraints are validated directly when you edit the DSL file with the DSL editor (error highlight). DSL constraints are defined in `SculptordslJavaValidator.java` and `SculptordslChecks.chk` in `org.sculptor.dsl`. If you change these you have to rebuild the plugins, but no other changes are needed.

Note that the constraints are both a help when editing the DSL and also a way to enforce certain design decisions.


### Meta Model Constraints

Constraints of the ecore meta model is located in `constraints.chk` in `sculptor-generator-core`.


## Transformations

[Xtend](http://www.eclipse.org/xtend/) is used for model transformations.


### DSL Model Transformation

Two meta models are used. One for the DSL, which is generated from the XText grammar. Another meta model is used by the code generation templates. They are rather similar, but still different. They serve different purposes, which motivates why two meta models are needed.

It would also be possible to replace the XText DSL meta model with some other, e.g. a graphical DSL implemented with [GMF](http://www.eclipse.org/modeling/gmp/).

A transformation converts the DSL model to the code generation model. This transformation is implemented in `DslTransformation.ext` in `sculptor-generator-core`.

You can do a lot of powerful things in this transformation, without having to change code generation templates or its meta model. For example, the [scaffold feature][2] is implemented in the transformation. There is a property `scaffold` for `DslEntity`. It is like an ordinary `Entity` with attributes and references, but the `DslTransformation` automatically add `Repository` and `Service` with generic CRUD operations.

When you change the DSL, except for simple syntactic sugar, you often have to change the `DslTransformation` also.


### Model Enrichment

There is one transformation that enrich the model with some useful default values. It is implemented in `Transformation.ext`. For example the id attribute of each DomainObject is added by this transformation.

You can add more features to this transformation when needed. If it is not specific to the concrete syntax of the DSL, it is better to add features to this transformation than to the `DslTransformation`, since they will be available even if the concrete syntax is replaced by some other DSL.


### GUI Transformation

A separate model is used for generation of the CRUD GUI. A transformation takes the domain model as input and and creates a model that is better suited for generation of the user interface. This transformation is located in `GuiTransformation.ext`


### Customize the Transformations

The [Sculptor extension mechanism](advanced-tutorial.html#overrides-and-extension-mechanism) can be used to customize the transformations.  Follow either the steps in the [overriding templates or transformations for a project](advanced-tutorial.html#overriding-templates-or-transformations-for-a-project) or [cartridges: reusable extensions to Sculptor](advanced-tutorial.html#cartridges-reusable-extensions-to-sculptor), depending on how you want to package the extension.

For example, to skip automatic addition of uuid property in your project:

* Locate the class to be overridden or extended, in this case it's the org.sculptor.generator.transform.Transformation class
* Create a TransformationOverride.xtend class in the generator package in a separate generator project for your main project
* Override the modifyUuid method:

~~~
package generator

import org.sculptor.generator.transform.Transformation
import org.sculptor.generator.chain.ChainOverride

@ChainOverride
class TransformationOverride extends Transformation {

  ...

  override void modifyUuid(DomainObject domainObject) {
  }
~~~


## Meta Model

Input to the code generation is the model, which is structured according to the meta models `sculptormetamodel.ecore` and `sculptorguimetamodel.ecore`. The meta models are defined with [Ecore](http://www.eclipse.org/modeling/emf/).


### Business Tier Model (domain model)

![Sculptormetamodel](/images/documentation/developers-guide/sculptormetamodel.png) \\
<small>_Figure 2. Meta Model for the code generation model_</small>

If you have installed Eclipse Graphical Modeling Framework SDK (Ganymede update site) you will be able to edit the ecore file with a graphical editor, see Figure 2. Open the `sculptormetamodel.ecore_diagram` with the editor.

To prevent OutOfMemoryError when you use the graphical ecore editor you can add `-XX:MaxPermSize=128m` in `eclipse.ini`, which is located in the Eclipse installation directory.
{: .alert }

When you add a totally new concept you have to add it to this meta model.
When you change the meta model it is not enough to build with maven. First you must generate EMF model code.

  1. An EMF genmodel is created from the ecore model. Normally you don't have to create a new genmodel, but when you do major changes you can remove the `sculptormetamodel.genmodel` and create a new from the ecore model using "File > New > Eclipse Modeling Framework > EMF Model".
  2. Open the genmodel and right click on the top node. Select Generate Model. It generates Java code to `src/main/java`. It is safe to remove that code and generate it again if needed.

Now you can package the meta model with `mvn install`.


### GUI Model

Sculptor doesn't aim at implementing a general purpose GUI model that can be used for any type of GUI. We are focusing on the kind of CRUD GUI illustrated in the [CRUD GUI Tutorial](crud-gui-tutorial).

![Sculptorguimetamodel](/images/documentation/developers-guide/sculptorguimetamodel.png) \\
<small>_Figure 3. GUI Meta Model for the GUI parts of the code generation model_</small>


**User Task**

The model is arranged around User Tasks, The term UserTask is inspired by the presentation [From User Story to User Interface](http://www.agile2007.org/index.php?page=sub/&id=430). In this presentation it is described that:

  * Tasks require intentional action on behalf of a tool's user
  * Tasks have an objective that can be completed
  * Tasks decompose into smaller tasks

Sculptor's implementation with [Spring Web Flow](http://www.springsource.org/spring-web-flow) is that a UserTask corresponds to Flow. A conversation corresponds to a top level UserTask and its subtasks.


## Code Generation Templates

[Xpand Language](http://www.openarchitectureware.org/pub/documentation/4.3.1/html/contents/core_reference.html#xpand_reference_introduction) is used for the templates.

The code generation starts in `Root.xpt`, which selects some course grained elements from the model and invokes other templates. In the templates you can access simple properties of the model objects and extension methods.

~~~
«DEFINE attributeTypeAndName FOR Attribute»
«getTypeName()» «name»
«ENDDEFINE»
~~~

The extension methods are defined with [Xtend](http://www.eclipse.org/xtend/) in `helper.ext`. Some methods are implemented inside this file with the [Expression Language](http://www.openarchitectureware.org/pub/documentation/4.3.1/html/contents/core_reference.html#r10_expressions_language) and some delegate to Java helper classes.

[Expression Language](http://www.openarchitectureware.org/pub/documentation/4.3.1/html/contents/core_reference.html#r10_expressions_language) is also used in the templates to for example select elements from the model.

~~~
references.select(r | !r.many && (r.referenceType.metaType == BasicType))
~~~


## How to

Some of the examples presented here are already implemented in Sculptor. It was convenient to add documentation and using real examples while implementing new features.


### How to exclude generation

For some special cases the default generation might not be appropriate and it is desirable to handle the special case with manual code instead. For example, assume we have a complex domain object and we need to do the JPA/Hibernate mapping manually.

This can be nicely solved with the Aspect-Oriented Programming features in Xpand. It is described in [Aspect-Oriented Programming in Xtend ](http://www.openarchitectureware.org/pub/documentation/4.3.1/html/contents/core_reference.html).

Define your advice in `src/main/resources/generator/SpecialCases.xpt`:

~~~
«IMPORT sculptormetamodel»
«EXTENSION extensions::helper»
«EXTENSION extensions::properties»

«REM»Skip createTable for Person«ENDREM»
«AROUND templates::db::OracleDDL::createTable FOR DomainObject»
    «IF name != "Person" -»
    	«targetDef.proceed() -»
    «ENDIF -»
«ENDAROUND»

«REM»Skip dropTable for Person«ENDREM»
«AROUND templates::db::OracleDDL::dropTable FOR DomainObject»
    «IF name != "Person" -»
    	«targetDef.proceed() -»
    «ENDIF -»
«ENDAROUND»

«REM»Skip DomainObject for Person«ENDREM»
«AROUND templates::db::DomainObject::domainObjectBase FOR DomainObject»
    «IF name != "Person" -»
    	«targetDef.proceed() -»
    «ENDIF -»
«ENDAROUND»

«AROUND templates::db::DomainObject::domainObjectSubclass FOR DomainObject»
    «IF name != "Person" -»
    	«targetDef.proceed() -»
    «ENDIF -»
«ENDAROUND»
~~~

Some of the generated artifacts can be controlled with properties in `sculptor-generator.properties`. Look in `default-sculptor-generator.properties` for properties named `generate.xxx`. For example, to skip generation of all junit tests:

~~~
generate.test=false
~~~


### How to customize persistence.xml

To adjust the JPA/Hibernate properties in `persistence.xml` you can add the following in `SpecialCases.xpt`:

~~~
«AROUND templates::jpa::JPA::persistenceUnitAdditionalProperties FOR Application»
    <property name="hibernate.show_sql" value="true" />
    <property name="hibernate.jdbc.batch_size" value="0" />
«ENDAROUND»
~~~

The above hibernate properties are useful when debugging.


### How to use own Hibernate User Type

[Hibernate User Type](http://docs.jboss.org/hibernate/core/3.6/reference/en-US/html/types.html#types-custom) is powerful when you need conversion between domain and database representations.

You can define your own type in sculptor-generator.properties and also define its hibernateType. An example is to define a Time type like this:

~~~
javaType.Time=org.joda.time.LocalTime
hibernateType.Time=org.joda.time.contrib.hibernate.PersistentLocalTimeAsString
db.oracle.type.Time=VARCHAR2
db.oracle.length.Time=14
~~~

This means that you can use `Time` in `model.btdesign` and ``@Type(type = "org.joda.time.contrib.hibernate.PersistentLocalTimeAsString")`` annotation is generated in the Java code.


### How to change package names in the generated code

In the DSL you can define the root package for the `Application`. You can define the package of a `Module`, if the default is not satisfactory. The package name of individual Domain Objects can be specified, if the default is not appropriate.

~~~
Application Library {
    basePackage = com.mycorp.library

    Module common {
        basePackage = com.mycorp.common

        BasicType Address {
            package=types
            String street
            String city
        }
    }
}
~~~

You can change the names of the subpackages to the module by defining properties in `sculptor-generator.properties` as described in [packageNames](#packageNames).


### How to add support for another database

Generation of database schema (DDL) is specific for different database vendors. To add support for your database you can do like this.

1. In the application specific `sculptor-generator.properties` you define the database properties for your database. Note that the `db.product` value must be `custom`.

   ~~~
db.product=custom
db.custom.maxNameLength=27
db.custom.hibernate.dialect=org.hibernate.dialect.OracleDialect
db.custom.onDeleteCascade=true
db.custom.type.Boolean=CHAR(1)
db.custom.type.boolean=CHAR(1)
db.custom.type.Integer=NUMBER
db.custom.length.Integer=10
db.custom.type.int=NUMBER
db.custom.length.int=10
db.custom.type.Long=NUMBER
db.custom.length.Long=20
db.custom.type.long=NUMBER
db.custom.length.long=20
db.custom.type.Date=DATE
db.custom.type.java.util.Date=DATE
db.custom.type.DateTime=DATE
db.custom.type.Timestamp=DATE
db.custom.type.BigDecimal=NUMBER
db.custom.type.Double=NUMBER
db.custom.type.double=NUMBER
db.custom.type.String=VARCHAR2
db.custom.length.String=100
db.custom.length.Enum=40
db.custom.type.discriminatorType.STRING=VARCHAR
db.custom.length.discriminatorType.STRING=31
db.custom.type.discriminatorType.CHAR=CHAR(1)
db.custom.type.discriminatorType.INTEGER=INTEGER
~~~

2. Create a template for the DDL generation and locate it in `templates/CustomDDL.xpt` in the classpath, before the `sculptor-generator-core` jar.

`OracleDDL.xpt` is a template you can mimic.

Contribution of support for Apache Derby has been posted in the
[Forum](http://www.mail-archive.com/fornax-developer@lists.sourceforge.net/msg00437.html)


### How to remove the circular dependency check

Define the following in `sculptor-generator.properties`:

~~~
check.cyclicDependencies=false
~~~


### How to add custom generic access object

The Sculptor runtime application framework provides some useful generic AccessObjects, such as `findById`, `findByQuery`, `save`. It is likely that you need to add other generic AccessObjects.

To add a new generic AccessObject you can do like this. Assume you need an AccessObject that can evict from second level cache.

**Evict AccessObject Interface:**

~~~ java
/**
 * This AccessObject evicts objects from second level cache.
 */
public interface EvictAccess<T> {

    /**
     * Evict persistent objects. If {@link #setId(Long) id} is
     * not specified all objects of this class
     * be evicted.
     * @param persistentClass class of the persistent object
     */
    public void setPersistentClass(Class persistentClass);

    /**
     * Evict a specific object. Set
     * {@link #setPersistentClass(Class) persistentClass}
     * also.
     * @param id the id of the object to evict
     */
    public void setId(Long id);

    /**
     * Evict query cache with specified cache region
     */
    public void setQueryCacheRegion(String queryCacheRegion);

    public void execute();

}
~~~

**Evict AccessObject Implementation:**

~~~ java
public class EvictAccessImpl<T> extends JpaAccessBase implements EvictAccess<T> {

    private Class persistentClass;
    private Long id;
    private String queryCacheRegion;

    public void setPersistentClass(Class persistentClass) {
        this.persistentClass = persistentClass;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public void setQueryCacheRegion(String queryCacheRegion) {
        this.queryCacheRegion = queryCacheRegion;
    }

    public void performExecute() throws PersistenceException {
        if (queryCacheRegion != null) {
            getHibernateSession().getSessionFactory().evictQueries(queryCacheRegion);
        }
        if (persistentClass != null) {
            if (id == null) {
                getHibernateSession().getSessionFactory().evict(persistentClass);
            } else {
                getHibernateSession().getSessionFactory().evict(persistentClass, id);
            }
        }
    }

}
~~~

There are a few conventions for the AccessObjects that the templates are based on:

  * There must be an `execute` method. In above example it is implemented in `JpaAccessBase`, which invokes `performExecute`.
  * There must be a `setEntityManager` method in the implementation. In above example it is implemented in `JpaAccessBase`.
  * Input parameters are defined in the interface as setter methods. These corresponds to the parameters of the repository method.
  * The result of the AccessObject must be available with `getResult` method, defined in the interface. This corresponds to the return type of the repository method. If void then no `getResult` is used.

Define the interface and implementation in `sculptor-generator.properties`:

~~~
framework.accessapi.EvictAccess=org.helloworld.common.accessapi.EvictAccess
framework.accessimpl.EvictAccessImpl=org.helloworld.common.accessimpl.EvictAccessImpl
~~~

The above addition of properties and classes means that you can use the evict AccessObject in your DSL model without any further changes of Sculptor, e.g.

**model.btdesign:**

~~~ java
Repository PlanetRepository {
    findByExample;
    evict(String queryCacheRegion);
    evict(Class persistentClass);
    evict(Class persistentClass, Long id);
}
~~~

This would result in the following generated code in the Repository `PlanetRepositoryBase`:

~~~ java
public void evict(String queryCacheRegion) {
    EvictAccess<Planet> ao = planetAccessFactory.createEvictAccess();
    ao.setQueryCacheRegion(queryCacheRegion);
    ao.execute();
}

public void evict(Class persistentClass) {
    EvictAccess<Planet> ao = planetAccessFactory.createEvictAccess();
    ao.setPersistentClass(persistentClass);
    ao.execute();
}

public void evict(Class persistentClass, Long id) {
    EvictAccess<Planet> ao = planetAccessFactory.createEvictAccess();
    ao.setPersistentClass(persistentClass);
    ao.setId(id);
    ao.execute();
}
~~~

The above is good enough in most cases, but it is also possible to define default parameters and return values for the RepositoryOperation, which means that you can skip those when using it in the DSL model. By this you can better support ["convention over configuration"](http://en.wikipedia.org/wiki/Convention_over_configuration).

For example assume that `evict(Class persistentClass)` is the most used operation and you would like that a simple `evict` declaration corresponds to this. Then you need to implement a `GenericAccessObjectStrategy`

~~~ java
public class EvictAccessObjectStrategy
        extends AbstractGenericAccessObjectStrategy
        implements GenericAccessObjectStrategy {

    public void addDefaultValues(RepositoryOperation operation) {
        if (operation.getParameters().isEmpty()) {
            addParameter(operation, "Class", "persistentClass");
        }
    }

    public String getGenericType(RepositoryOperation operation) {
        DomainObject aggregateRoot = operation.getRepository().getAggregateRoot();
        String aggregateRootName = GenerationHelper.getPackage(aggregateRoot) + "." +
            aggregateRoot.getName();
        return "<" + aggregateRootName + ">";
    }

    public boolean isPersistentClassConstructor() {
        return false;
    }

}
~~~

and register it in `sculptor-generator.properties`:

~~~
framework.accessapi.EvictAccess=org.helloworld.common.accessapi.EvictAccess
framework.accessimpl.EvictAccessImpl=org.helloworld.common.accessimpl.EvictAccessImpl
genericAccessObjectStrategy.evict=org.helloworld.common.accessimpl.EvictAccessObjectStrategy
~~~

To understand how to implement the strategy class you can have a look in `GenericAccessObjectManager` and the strategies implemented for the ordinary generic AccessObjects. It is located in `sculptor-generator-core`. These strategies are only used by the code generator. They are not used in runtime.

The above means that you can use `evict` without any parameters in the DSL model `model.btdesign`.

~~~
Repository PlanetRepository {
    findByExample;
    evict;
}
~~~

and the generated Repository method would look like:

~~~ java
public void evict(Class persistentClass) {
    EvictAccess<Planet> ao = planetAccessFactory.createEvictAccess();
    ao.setPersistentClass(persistentClass);
    ao.execute();
}
~~~

For some rare special cases you might need to adjust the code generation template for the Repository, `genericBaseRepositoryMethod` in `Repository.xpt`.

You must place the `genericAccessObjectStrategy` class (EvictAccessObjectStrategy) in another library than your application project. It is used (only) by the code generator, and that is run before ordinary compilation, i.e. the class is not available in generate-sources phase if a clean has been done before.
{: .alert}


### How to add a transformation feature

[Scaffold][2] is a feature to be able to mark a Domain Object as scaffold and then automatically add some predefined operations (typically CRUD) to the corresponding Repository and Service. This is implemented in the transformation.

We add a boolean `scaffold` property to the DSL grammar for DslEntity. This is straightforward.

In the transformation `DslTransformation.ext` we add a Repository and Service if they doesn't already exist. Operations are added to these, if they don't exist.

~~~
scaffold(sculptormetamodel::DomainObject domainObject) :
    (domainObject.repository == null ?
        domainObject.addRepository() :
        null) ->
    domainObject.repository.addScaffoldOperations() ->
    (domainObject.module.services.exists(s | s.name == (domainObject.name + "Service")) ?
        null :
        domainObject.module.addService(domainObject.name + "Service")) ->
    domainObject.module.services.select(s | s.name == (domainObject.name + "Service")).
            addScaffoldOperations(domainObject.repository);
~~~

It is easiest to implement the actual creation and addition of model elements in Java. We implement the methods `addRepository`, `addService` and `addScaffoldOperations` in a Java helper class.

~~~
public static DomainObject addRepository(DomainObject domainObject) {
    SculptormetamodelFactory factory = SculptormetamodelFactoryImpl.eINSTANCE;

    Repository repository = factory.createRepository();
    repository.setName(domainObject.getName() + "Repository");
    domainObject.setRepository(repository);

    return domainObject;
}
~~~


### How to change auditable column names

The auditable properties are added by a model transformation. You can modify that transformation in this way.

Define your advice in `src/main/resources/generator/SpecialCases.ext`.

~~~
import sculptormetamodel;

around transformation::Transformation::addAuditable(Entity entity) :
         addAuditDateAttribute(entity, "createdDate", "created_date") ->
         addAuditByAttribute(entity, "createdBy", "created_by") ->
         addAuditDateAttribute(entity, "lastUpdated", "updated_date") ->
         addAuditByAttribute(entity, "lastUpdatedBy", "updated_by");

create Attribute addAuditDateAttribute(Entity entity, String propertyName, String databaseName) :
setName(propertyName) ->
setDatabaseColumn(databaseName) ->
setType("java.util.Date") ->
setNullable(true) ->
         entity.attributes.add(this);

create Attribute addAuditByAttribute(Entity entity, String propertyName, String databaseName) :
setName(propertyName) ->
setDatabaseColumn(databaseName) ->
setType("String") ->
setLength("50") ->
setNullable(true) ->
         entity.attributes.add(this);
~~~


### How to change syntactic sugar

It is rather easy to change the concrete syntax of the DSL. It is defined in the [XText](http://www.eclipse.org/Xtext/) grammar `sculptordsl.xtext`.

For example, as an alternative to `!` we would like to be able to use `not`.

Define a rule.

~~~
terminal NOT :
  ('!'|'not');
~~~

Use this rule instead of the "!" literal.

~~~
((notChangeable?=NOT "changeable") | ("changeable")) |
~~~

If you change the core elements of the language, such as `DslService`, `DslEntity`, `DslAttribute`, you have to change the transformation also, which is located in `DslTransformation.xpt` in the ´sculptor-generator-core`` project.

**Try DSL changes**

When working with the DSL you generate the parser and editor by running the `GenerateSculptordsl.mwe` in the
`org.sculptor.dsl` project (Run as MWE Workflow). Thereafter you can try the DSL editor by launching a runtime workbench as an Eclipse application, which includes the plugins in the workspace automatically.


### How to add a new feature in the meta model

We need to be able to define an additional feature for the attributes of the Domain Objects. It should be possible to define that an attribute is included in the constructor, but still have setter method. A complement to the changeable feature. Let us call this new feature `required`.

1. Open the `sculptormetamodel.ecore_diagram` with the graphical editor.
   Add a new EAttribute to the `Attribute` "class". Set the name to `required` in the Properties view. Select `EBoolean` in the `EAttribute Type`.

   ![Required Attribute](/images/documentation/developers-guide/required_attribute.png) \\
   <small>_Figure 4. Screenshot of metamodel for required attribute_</small>

2. Open the `sculptormetamodel.genmodel` and right click on the top node. Select Generate Model.

3. Open `sculptordsl.xtext` and add `required` in the same way as `nullable` in the `DslAttribute` section. Note that `required` should by default be treated as false when not specified.

4. Run `GenerateSculptordsl.mwe` in `org.sculptor.dsl` project. This can be done by right clicking `GenerateSculptordsl.mwe` and selecting "Run as MWE Workflow".

5. Open `DslTransformation.ext` and add the "copy" of the `required` property in the same way as the `nullable` property in the create Attribute method.

   ~~~
setRequired(attribute.required) ->
   ~~~

6. Modify the code generation templates. In this case it was only needed to adjust the logic for the `constructorAttributes` method, which is located in `helper.ext`.

7. Build with `mvn clean install`.

8. Install the DSL editor plugins and test the new feature with the reference application.


### How to Add a New Core Concept

This section describes all steps of how to add a completely new concept, Consumer in this case. Consumer is implemented in Sculptor and since this description is very brief you have to look at the details in the source code to make any sense out of this.

1. Start with the Ecore meta model. Open the `sculptormetamodel.ecore_diagram` with the graphical editor.

   * Add `Consumer` class.
   * Add generalization link from `Consumer` to `NamedElement`, a Consumer has a name.
   * Add bi-directional aggregate association between `Module` and `Consumer`, one Module can contain many Consumers, a Consumer belongs to one Module.
   * Add association from `Consumer` to `Service`, `serviceDependencies`, used for dependency injection of Services into the Consumer.
   * Add association from `Consumer` to `Repository`, `repositoryDependencies`, used for dependency injection of Repositories into the Consumer.

   ![Consumer Meta Model](/images/documentation/developers-guide/consumer_meta_model.png)\\
   <small>_Figure 5. Screenshot of metamodel for consumer_</small>

2. DSL grammar, located in `sculptordsl.xtext`

   * Define `DslConsumer`

     ~~~
DslConsumer :
      (doc=STRING)?
      "Consumer" name=ID "{"
        (dependencies+=DslDependency)*
        ("unmarshall" "to" (("@")?messageRoot=ID))?
        ("queueName" "=" queue=DslQueueIdentifier )?
      "}";
     ~~~

   * Add `(consumers+=DslConsumer)*` to `DslModule`
   * Add `DslDependency` in the same way as the existing `DslRepositoryDependency`
   * Run `GenerateSculptordsl.mwe2` in `org.sculptor.dsl` to generate DSL meta model and DSL editor

3. DSL constraints checks, located in `SculptordslJavaValidator.java` and `SculptordslChecks.chk`.

   * Add `DslServiceDependency` in the same way as the existing `DslRepositoryDependency`, `findService` is already implemented in `sculptordsl.ext`

4. Transformation of DSL model to Ecore meta model, located in `DslTransformation.ext` in `sculptor-generator-core`.

   * Add `consumers.addAll(module.consumers.transform()) ->` to the `Module` transformation.
   * Add `create sculptormetamodel::Consumer this transform(DslConsumer consumer)...`
   * Add `module.consumers.transformDependencies() ->` to the `Module` transformation.
   * Add `transformDependencies(DslConsumer consumer)...`
   * Add `sculptormetamodel::Service transformServiceDependency(DslDependency dependency)...`

5. Meta model constraints, located in `constraints.chk` in `sculptor-generator-core`.

   * Add `context Consumer ERROR "Not allowed to delegate to repository...`
   * Add check of cyclic dependencies in `DependencyConstraints` class

6. Code generation template

   * Add `Consumer.xpt` and everything to generate for the Consumers.
   * Add properties for package and framework classes. This is done in `default-sculptor-generator.properties`, `properties.ext` and `helper.ext`.
   * Add `EXPAND Consumer` in `Root.xpt`, you need `allConsumers()` in `helper.ext`, which can be implemented in the same way as `allServices`.
   * Add Spring stuff for the Consumers in `Spring.xpt`.

### How to define a Sculptor cartridge

Cartridges in Sculptor provide a means to package extensions to Sculptor that projects may enable or disable via the 'cartridges' property.  Some features within Sculptor itself are packaged as cartridges, for example the builders feature and MongoDB support.

To define a cartridge:

* Review [overriding templates or transformations for a project](advanced-tutorial.html#overriding-templates-or-transformations-for-a-project) and [cartridges: reusable extensions to Sculptor](advanced-tutorial.html#cartridges-reusable-extensions-to-sculptor)
* Follow the steps in [cartridges: reusable extensions to Sculptor](advanced-tutorial.html#cartridges-reusable-extensions-to-sculptor) to define a cartridge.
* The cartridge may either be packaged in the sculptor-generator-core project itself, or in a separate project.

  If a separate project, it can be set up in the same manner as [Creating separate generator project](advanced-tutorial.html#creating-separate-generator-project)



[1]: development-environment
[2]: advanced-tutorial#scaffold
[3]: maven-plugin
[4]: advanced-tutorial#auditable
