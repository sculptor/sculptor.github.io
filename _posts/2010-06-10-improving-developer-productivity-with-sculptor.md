---
layout: post
title: "Improving Developer Productivity with Sculptor"
description: ""
navbar_name: blog
category: 
tags: [DDD, DSL, Sculptor]
author: Patrik Nordwall
---
{% include JB/setup %}

*Do you still code everything by hand? Isn't it tedious and error prone? It's time to start using Sculptor to jump start Model Driven Software Development. Concepts and patterns from Domain-Driven Design are used in the Domain Specific Language, which is the model for the generated Hibernate and Spring implementation. Have you had frustrating experiences with code generators? Have you become disillusioned with code generators? Was the generated code not entirely satisfactory, and you couldn't control the result? Sculptor is different!*

* toc
{:toc}


## Overview

Sculptor is a simple and powerful code generation platform, which provides a quick start to [Model Driven Software Development](https://www.voelter.de/mdsd-book/) (MDSD). When using Sculptor you can focus on the business domain, instead of technical details. You can use the concepts from [Domain-Driven Design](https://domaindrivendesign.org/books/) (DDD) in the textual Domain Specific Language (DSL). Sculptor uses [Xtext](https://www.eclipse.org/Xtext/) to parse the DSL and generate high quality Java code and configuration. The generated code is based on well-known frameworks, such as Spring, Hibernate and Java EE.

![Overview](/images/2010-06-10-improving-developer-productivity-with-sculptor/sculptor_overview5.png) \\
<small>_Figure 1. Overview of Sculptor_</small>

Figure 1, “Overview” illustrates how the developer specifies the application design in the DSL, generates code and configuration with Maven 2. Generated code and hand written code is well separated. Hand written code, such as JUnit tests and business logic, is added in subclasses or other well defined places.

The DSL and the code generation drives the development and is not a one time shot. It is an iterative process, which can be combined with Test Driven Development (TDD) and evolutionary design, as explained in the chapter [Test Driven Development with Sculptor](#tdd).

High level features of Sculptor:

  * Possibility to use the concepts from Domain-Driven Design (DDD) directly in the program language (the textual DSL). E.g. Module, Service, Entity, Value Object, Repository, ...

  * It provides a "best practice design" for Hibernate and Spring. The target environment is real enterprise systems. It is easy to make CRUD-services, but the design is based on the assumption that more than simple CRUDs are needed (flexibility, business logic, ...).

  * The design and the generated code is better and more complete than existing code generation tools (e.g. Hibernate synchronizer, Hibernate reveng) can achieve out of the box. Maybe the biggest advantage is that a complete application can be generated from a single model, not only fragments that are hard to fit in to the overall design.

  * Quick start. The initial investment for getting started with MDSD can be big and this platform reduces that a lot.

  * Sculptor is not a one-size-fits-all product. Even though it is a good start for many systems, sooner or later customization is always needed. Sculptor is designed and documented with this in mind. The generated result can easily be modified to meet your needs.

To illustrate how Sculptor can be used in practice we will use an example. It is an introduction and not a complete User's Guide, more information can be found in the [Sculptor Documentation][1].

The example is a simple system for a library of movies and books. The core of the system is a Domain Model, see Figure 2, “Domain model” A Library consists of PhysicalMedia. Books and Movies are different types of Media, which are stored on a PhysicalMedia, e.g. DVD, VHS, paper books, eBooks on CD. A Media has Characters, e.g. James Bond, which can be played by a Person, e.g. Pierce Brosnan. A person can be involved (Engagement) in different Media, actually a Person can have several Engagements in the same Media. E.g. Quentin Tarantino is both actor and director in the movie 'Reservoir Dogs'.

![UML Graph](/images/2010-06-10-improving-developer-productivity-with-sculptor/umlgraph.png) \\
<small>_Figure 2. Domain model of the Library example._</small>

With a few simple Maven commands you will be able to create the Maven and Eclipse projects for your application. Sculptor provides [Maven Archetypes][2] to facilitate this.

A Sculptor application is defined in a textual DSL. Since it is text it has all the benefits of ordinary text source code, such as searching, copy-paste, merging and so on. Sculptor provides an Eclipse editor for the DSL. It supports error highlight, code completion and outline view.

Sculptor doesn't mandate any specific development methodology, but personally I prefer Test Driven Development with evolutionary design and refactoring. Using that approach the DSL model is not a big design up front. This is described in more detail in the chapter [Test Driven Development with Sculptor](#tdd).

An early DSL model for the Library example might look like this:

~~~ java
Application Library {
  basePackage = org.library

  Module movie {

    Service LibraryService {
      findLibraryByName delegates to LibraryRepository.findLibraryByName;
      saveLibrary delegates to LibraryRepository.save;
      findMovieByName delegates to MovieRepository.findByName;
    }

    Entity Library {
      String name key
      reference Set movies

      Repository LibraryRepository {
        save;
        @Library findLibraryByName(String name) throws LibraryNotFoundException;
        protected findByCriteria;
      }
    }

    Entity Movie {
      String title not changeable
      String urlIMDB key
      Integer playLength

      Repository MovieRepository {
        List findByName(Long libraryId, String name)
        delegates to FindMovieByNameAccess;
      }
    }
  }
}
~~~

The full DSL model for the Library example looks like [this][3]. There you can see that the DSL defines two Modules, containing a few Services. It defines the same Domain Objects, including attributes and references, as in Figure 2, “Domain model”.

The core concepts of the DSL have been taken from [Domain-Driven Design](https://domaindrivendesign.org/books/). If you don't have the book you can download and read more in [DDD Quickly](https://www.infoq.com/news/2006/12/domain-driven-design).

Sculptor code generation is executed as part of the [Maven build cycle](https://maven.apache.org/guides/introduction/introduction-to-the-lifecycle.html), i.e. you run `mvn install` as usual. When developing it is also convenient to use `mvn generate-sources`, which will only generate, without performing compile, test and package.

The artifacts that are intended to be completed with hand written code are generated only once, while other artifacts are regenerated and overwritten each time. Separation is typically implemented with extension as illustrated in Figure 3, "Separation of generated and hand written".

![Sculptor Separation of generated and hand written](/images/2010-06-10-improving-developer-productivity-with-sculptor/sculptor-separation.png) \\
<small>_Figure 3. Separation of generated and hand written_</small>

The regenerated source should not be added to version control. These two types of artifacts are separated from each other in the file system. The source and resources directories can be seen in Figure 4, “File structure”. It also shows the packages and generated classes in the domain package for the media module. The names of the packages can of course be changed easily.

![Filestructure](/images/2010-06-10-improving-developer-productivity-with-sculptor/filestructure.png) \\
<small>_Figure 4. File structure and packages_</small>

One of the strengths of Sculptor, compared to other code generators, is that it spans the complete application, not only fragments, which are hard to fit in to the overall design.

Sculptor typically reduce the amount of hand written code with 50% compared to a manually coded application with a similar design. The following section presents measurements from a real case.


### Sculptor Metrics

I have converted an existing application to use Sculptor. It is interesting to see that Sculptor removes about half of the code.

It is a rather small application with a straight forward search and display user interface. It also contains an administrative part with typical CRUD operations. The system collects data from a other systems via messages from the ESB. It consists of 2 Services, 6 Consumers, 7 persistent Domain Objects, and several non persistent Value Objects. In these metrics we have not included the presentation tier, since Sculptor doesn't provide anything for the presentation tier yet.

The design of the old and the converted application is very similar, except for the introduction of Repositories and Access Objects instead of ordinary Data Access Objects (DAO).

![Metrics](/images/2010-06-10-improving-developer-productivity-with-sculptor/metrics.png) \\
<small>_Figure 5. Lines of code for old and converted application_<small>

Figure 5, “Lines of code” shows the lines of codes for the old application compared to the new converted application, which utilize Sculptor.
The first bar in each category is the old application, the second is the generated code by Sculptor, and the third bar is hand written code in the new application.

The percentage measures are the amount hand written code (skeleton generated once) in the new application related to the amount of code in the
old application, which was hand written. In total about 50% of the code is generated by Sculptor for this application.

Reduction of lines of code is definitely not the only benefit of Sculptor. Quality aspects such as consistency and maintainability are also
important, but not as easy to measure.


### Domain Objects

In the context of Sculptor, Domain Object is a common term for Entity, ValueObject and BasicType.

Entities have an identity and the state of the object may change during the lifecycle. For Value Objects the values of the attribues are interesting, and not which object it is. Value Objects are typically immutable. BasicType is used for fundamental [types](https://www.martinfowler.com/ieeeSoftware/whenType.pdf), such as [Money](https://martinfowler.com/eaaCatalog/money.html). BasicType is a ValueObject which is stored in the same table as the Domain Object referencing it. It corresponds to JPA embedded (Hibernate Component).

Domain Objects are implemented as ordinary POJOs or EJB3 entities. They can have simple attributes and references to other Domain Objects. They can of course also contain behavior; otherwise it wouldn't be a rich domain model. However, the behavior logic is always written manually and not defined in the DSL.

It is possible to specify that a ValueObject is not persistent, i.e. not stored in database. This is for example useful for parameters and return values for service operations. It is also possible to define plain DataTransferObjects, which are typically used for external services as web services.

Below is the definition in the DSL of a few Domain Objects in the Library example.

~~~ java
Entity Person {
  String ssn key length="20"
  String country key length="2"
  Integer age
  reference @PersonName name
}

BasicType PersonName {
  String first
  String last
}

ValueObject MediaCharacter {
  String name not changeable
  reference Set playedBy
  reference Set existsInMedia opposite characters

}
~~~

For Domain Objects Sculptor generates:

  * Data part of the domain objects. 
  * JPA and Hibernate annotations for the attributes and associations of the domain model. 
  * JAXB annotations for DTOs
  * Constructors and accessor methods that reflect the changeability of the object. 
  * Hibernate validator annotations
  * equals, hashCode and toString 
  * Meta-data for properties to be used with findByCondition
  * Database definition script. 
  * Visualization class diagram
  * Summary documentation in clickable HTML format

Separation of generated and manually written code is done by a generated base class and manually written subclass. See Figure 3, "Separation of generated and hand written". It is in the subclass you add methods to implement the behavior of the Domain Object. The subclass is also generated, but only once, it will never be overwritten by the generator. You can of course remove it to regenerate it.

`equals` and `hashCode` requires some thought when used with Hibernate, see the [discussion](https://www.hibernate.org/109.html) at the Hibernate site. Sculptor takes care of the details and this is a typical example of how Sculptor raises the level of abstraction by distilling best practice. You only have to mark the attributes that is the natural key of the Domain Object, or if there is no natural key Sculptor will generate a UUID automatically.

The Sculptor DSL is based on the principle "convention over configuration". An example of this is that Entities are by default auditable, which means that when the objects are saved an interceptor will automatically update information about by whom and when the object was created or changed These attributes are automatically added for auditable Domain Objects. You can turn off auditing for an Entity with `not auditable`. Value Objects are by default not auditable.


### Services

The Services act as a [Service Layer](https://martinfowler.com/eaaCatalog/serviceLayer.html) around the domain model. It provides a well defined interface with a set of available operations to the clients.

Services are implemented as Spring @Service with interface and implementation class. EJB 3 stateless session bean is also supported.

The transaction boundary is at the service layer. JPA/Hibernate session demarcation and error handling are implemented with Spring AOP in front of the Services.

In the DSL an operation of a Service almost looks like an ordinary Java method with return type, parameters and throws declaration.

~~~
Service LibraryService {
  inject LibraryRepository
  inject MediaRepository
  @Library findLibraryByName(String name)
    throws LibraryNotFoundException;
  saveLibrary delegates to LibraryRepository.save;
  findMediaByName delegates to MediaRepository.findMediaByName;
  List findMediaByCharacter(Long libraryId,
    String characterName);
  findPersonByName delegates to PersonService.findPersonByName;
}
~~~

For Services Sculptor generates:

  * Service boilerplate code, interface and implementation
  * Delegation mechanism to Repositories and other Services.
  * JAX-WS Web Services
  * Data Transfer Objects (DTO) with JAXB annotations
  * Consumer boilerplate code
  * Error handling, logging
  * Spring annotations and XML configuration
  * Starting points for JUnit tests
  * Visualization class diagram
  * Summary documentation in clickable HTML format

In the Service implementation hand written code can be added for the behavior of the Service. You can also easily delegate to an operation in a Repository or another Service. Then you only have to declare the name of the operation in the DSL. Return type and parameters are "copied" from the delegate operation.

Sculptor can also generate message consumers, implemented as EJB Message Driven Beans for the pure EJB3 target implementation.


### Repositories

The Repositories encapsulate all technical details for retrieving Domain Objects from the database. They are also used to make new objects persistent and to delete objects.

The interface of the Repository speaks the Ubiquitous Language of the domain. It provides domain centric data access of the Aggregate roots of the domain model.

~~~
Repository MediaRepository {
  int getNumberOfMovies(Long libraryId) delegates to AccessObject;
  save;
  findByQuery;
  List findMediaByName(Long libraryId, String name);
}
~~~

For Repositories Sculptor generates:

  * Repository boilerplate code
  * Spring annotations and XML configuration
  * Generic operations such as: 
    * findById
    * findAll
    * findByKey
    * findByQuery
    * findByCondition
    * save
    * delete

The default implementation of a Repository consists of an implementation class and Access Objects. The intention is a separation of concerns between the domain and the data layer. Repository is close to the business domain and Access Objects are close to the data layer. The JPA/Hibernate specific code is located in the Access Object, and not in the Repository. You can chose to skip this separation and write everything in the repository.

Sculptor runtime framework provides generic Access Objects for operations such as `findByCondition`, `findByQuery`, `findByKeys`, `save` and `delete`. It is important that the API of the Repository communicates supported operations for the Aggregate root and therefore these generic operations are not added automatically to the Repository. They must be specified in the DSL, but that is very simple.


## Test Driven Development with Sculptor
{: #tdd}

Sculptor doesn't mandate any specific development methodology, but personally I prefer Test Driven Development with evolutionary design and refactoring.
It is important to explain that Sculptor can be combined with an agile mindset and Test Driven Development (TDD) in particular.
Using this approach the DSL model is not a big design up front.

Unit tests of Sculptor artifacts can be done for Domain Objects, Repositories, Services and Consumers.

Domain Objects can be tested as ordinary POJOs with JUnit, exactly as usual. It is probably the behavior that needs to be tested and methods for that are added manually anyway.

When testing Repositories, Services and Consumers it is convenient to use [DBUnit](https://www.dbunit.org/) to load the in memory HSQLDB database with test data from an XML file.
The database is recreated for each test method.

Sometimes it is better to test the Services and Consumers by [stubbing](https://www.martinfowler.com/eaaCatalog/serviceStub.html) dependencies to Repositories and other Services.

To explain the TDD approach we will look at an example. It is the same example application as described in this article.
Assume we need to add a service to lookup a library with a specific name.

1. Add test method. Write asserts and call the non-existing service method.

   ~~~ java
	public void testFindLibraryByName() {
	    String name = "famous";
	    Library library = libraryService.findLibraryByName(
	            getServiceContext(), name);
	    assertNotNull(library);
	    assertEquals(name, library.getName());
	}
   ~~~

2. Add test data in the DBUnit XML file.

   ~~~ xml
	<?xml version='1.0' encoding='UTF-8'?>
	<dataset>
		<LIBRARY ID="1" NAME="famous" VERSION="1"/>
	</dataset>
   ~~~     

3. Thereafter you have a good feeling of the API of the method. Add it to the DSL model.

   ~~~ java
	Service LibraryService {
	    @Library findLibraryByName(String name);
	    saveLibrary delegates to LibraryRepository.save;
	    findMediaByName delegates to MediaRepository.findMediaByName;
	    findMediaByCharacter delegates to
	      MediaRepository.findMediaByCharacter;
	    findPersonByName delegates to PersonService.findPersonByName;
	}
   ~~~     

4. Generate code with with `mvn generate-sources`. When you add new methods it sometimes results in compilation errors in the hand written classes,
   which are only generated once. In this case `LibraryServiceImpl`.
   These types of compilation errors can easily be fixed with Eclipse `ctrl+1 Add unimplemented methods`.

5. Now the compilation error in the test class has disapeared. Run the test to see red bar.

6. Implement the method, by adding hand written code and/or adding more stuff to the DSL model.
   In this case we add a generic `findByCondition` operation in the Repository. We also add a dependency injection of the Repository in the Service.

   ~~~ java
	Service LibraryService {
	    inject LibraryRepository
	    @Library findLibraryByName(String name);
	    saveLibrary delegates to LibraryRepository.save;
	    findMediaByName delegates to MediaRepository.findMediaByName;
	    findMediaByCharacter delegates to
	      MediaRepository.findMediaByCharacter;
	    findPersonByName delegates to PersonService.findPersonByName;
	}
	
	Entity Library {
	    String name key
	    reference Set media opposite library
	
	    Repository LibraryRepository {
	      findByCondition;
	      findById;
	      save;
	      findByQuery;
	    }
	}
   ~~~

   Add some hand written code in `LibraryServiceImpl` to call `findByCondition`.

   ~~~ java
	public Library findLibraryByName(ServiceContext ctx, String name) {
	    List criteria =
	        criteriaFor(Library.class).withProperty(name()).eq(name).build();
	    List libraries = getLibraryRepository().findByCondition(criteria);
	    return libraries.get(0);
	}
   ~~~

7. Run test. Green bar!

8. Continue in the same way by adding another test method, e.g. for the failure scenario when a matching library is not found.

9. Refactor if necessary.


### Refactoring

Refactoring can be done like this. Use the ordinary refactoring tools in Eclipse, which will modify the generated code also.
Run tests to make sure you still have green bar. Thereafter you do the corresponding changes in the DSL model and generate with `mvn generate-sources`. Run tests.

Before generating, all files in the generated directories can be removed to make sure that everything is in sync.

An alternative approach, which is better in some cases, is to start with changing the DSL model and regenerate.
Thereafter you manually change or move the hand written code.


## CRUD GUI

The rich domain model is the core piece of Sculptor, but Sculptor also provides nice front-end implementations. The purpose of the front-end is to manage the classical Create, Read, Update, and Delete operations of the domain objects. It also has advanced support for associations between objects.

There are three different implementations you can choose between. 

  * Web Application with JSF and Spring WebFlow
  * Rich Internet Application with GWT Smartclient
  * Rich Client with Eclipse RCP

![Screenshot of Smartclient GUI](/images/2010-06-10-improving-developer-productivity-with-sculptor/screen1.png?height=253&width=400) \\
<small>_Figure 6. Screenshot of Smartclient GUI_</small>


## Customization

This section is intended as a teaser of how Sculptor can be customized. More information is available in the [Sculptor Documentation][1].

Sculptor is not a one-size-fits-all product. Even though it is a good start for many systems, sooner or later customization is always needed. Some things can easily be changed with properties or AOP, while other things require more effort. However, Sculptor doesn't pretend that it can solve all problems out-of-the-box, but is designed and documented so that you as a developer can take full control of the tool without too much effort.


### Pick 'n' Chose Target Implementation

The default target implementation is using:

  * JPA with Hibernate as provider
  * Spring with annotations, no EJBs
  * Web CRUD GUI client with JSF, Facelets, and Spring Web Flow
  * In-memory Hsqldb as database
  * Deployment as war in Jetty

The default choices gives a good starting environment with minimal requirements of installation of external software, such as database and application server.

For each of these areas there are alternative implementations that are supported by Sculptor out of the box. The features are selected with simple configuration.

![Sculptor Target Implementations](/images/2010-06-10-improving-developer-productivity-with-sculptor/sculptor-target-impl.png) \\
<small>_Figure 7. Target Implementations_</small>


### Internal Design

Sculptor is implemented with [Xtext](https://www.eclipse.org/Xtext/) and [Xpand](https://wiki.eclipse.org/Xpand). Sculptor is Open Source and available under Apache 2 License.

![Sculptor Internal Design](/images/2010-06-10-improving-developer-productivity-with-sculptor/sculptor-internal-design.png) \\
<small>_Figure 8. Internal Design of Sculptor_</small>

  1. The developer is using the DSL Editor plugin to edit the application specific `model.btdesign`, i.e. the source for the concrete model that is the input to the code generation process. Constraints of the DSL are validated while editing.

  2. When generating code the application specific `workflow.oaw` is executed. It doesn't contain much.

  3. It invokes the `sculptorworkflow.oaw` which defines the flow of the code generation process.

  4. It starts with parsing the `model.btdesign` file using the XText parser. Constraints of the DSL are checked.

  5. The DSL model is transformed into a model of the type defined by the `sculptormetamodel.ecore` meta model. The meta model is defined with EMF Ecore.

  6. Constraint validation is performed.

  7. The model is transformed again. Now it is actually modified to add some default values.

  8. Constraint validation again.

  9. Now the actual generation of Java code and configuration files begins. It is done with code generation templates written in the XPand language. The templates extract values from the model and uses Xtend and Java helper classes.

  10. Properties of technical nature, which don't belong in the DSL or meta model, are used by the templates and the helpers.

You can customize rather much with simple configuration properties. Examples:

  * Replace the whole, or parts, of the runtime framework.

  * Add custom generic Access Objects.

  * Select database product. MySQL, Oracle and PostgreSQL are supported out-of-the-box, but other databases can be added easily.

  * Define type mapping from DSL types to database and Java types.

  * Package naming.

  * JPA/Hibernate support directly in Repository, instead of Access Objects.


### Change Generation Templates

The actual code generation is done with [XPand](https://wiki.eclipse.org/Xpand), which is a powerful and simple to use template language. The templates can be structured very much like methods, with small definitions, which expand other definitions.

You can change the code generation templates using the Aspect-Oriented Programming (AOP) feautures in XPand. You can "override" the definitions in the original templates. For example if you need to replace the UUID generation:

~~~
«IMPORT sculptormetamodel»
«EXTENSION extensions::helper»
«EXTENSION extensions::properties»

«AROUND templates::DomainObject::uuidAccessor FOR DomainObject»
    public String getUuid() {
        // lazy init of UUID
        if (uuid == null) {
            uuid = org.myorg.MyUUIDGenerator.generate().toString();
        }
        return uuid;
    }

    private void setUuid(String uuid) {
        this.uuid = uuid;
    }
«ENDAROUND»
~~~

It is also possible to use AOP to exclude generation. For some special cases the default generation might not be appropriate and it is desirable to handle the special case with manual code instead. For example, assume we have a complex domain object and we need to do the JPA/Hibernate mapping manually.


## Full Control

You are not stuck if you need to do more customization than what is possible with properties and AOP. You can checkout the Sculptor source code to do more adjustments. It is well described in the [Sculptor Documentation][1] how to change different things. A few examples of stuff you can modify:

  * Syntax of the DSL.

  * Meta model to add new language elements.

  * Code generation templates to generate new artifacts.

  * Constraint checks to enforce certain design decisions.


## Conclusions

Sculptor is an Open Source tool with the purpose to improve developer productivity and quality.
It raises the level of abstraction and automates a lot of otherwise repetitive manual coding.

You can focus on the business domain, instead of technical details. Sculptor distills best practice into easy to use notation.
The design of the generated application is heavily inspired by the patterns and concepts from [Domain-Driven Design](https://domaindrivendesign.org/books/).

You are in control and can easily adopt the tool to fit the requirements and frameworks you are working with.


   [1]: /documentation/
   [2]: /documentation/maven-archetypes
   [3]: /documentation/advanced-tutorial#library
