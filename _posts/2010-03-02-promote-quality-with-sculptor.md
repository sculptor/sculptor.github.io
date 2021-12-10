---
layout: post
title: "Promote Quality with Sculptor"
description: ""
navbar_name: blog
category: 
tags: [DDD, DSL, Sculptor]
author: Patrik Nordwall
---
{% include JB/setup %}

**Without a vision of how to design applications within an organization the development can be compared to lawless Wild West. Development guidelines are often used, but seldom successful over the long haul. We suggest taking the architectural decisions one step further by automating them using a tool such as Sculptor. The goal of Sculptor Open Source tool is to make us developers more productive and produce better quality software. This article will focus on the quality aspect and more precisely on how to enforce best practices and consistency.**

When using a general purpose language, such as Java, and its big toolbox of APIs and frameworks there is a huge freedom of choice. This is a double-edged sword. We meet a lot of companies that have realized that they must narrow down the choices so that each new project doesn't invent its own unique system architecture and product suite. The benefits of a homogeneous architecture is obvious when looking at the big picture, for instance:

  * Better utilization of developer competence.
  * Quicker start-up and development.
  * Easier to understand when changing tasks. The system typically have a long life cycles and developers in the maintenance organization work with several systems.

When doing this kind of standardization it is important to understand that not all systems have the same characteristics and should not be forced to use the same one-size-fits-all solution. It must be some freedom and choices depending on the nature of the system. It must be a balance between consistency and alternatives.

The reference architecture is often accomplished by writing guidelines and maybe a sample reference application. There are several problems with a reference architecture that is only promoted by documentation:

|---------------------------------+------------------------------|
| Problem with documentation only | Solution when using Sculptor |
|:------------------------------- | :----------------------------|
| Different developers have different interpretation of how to implement the suggested design patterns. | The code generator is always consequent. |
| Patterns are often implemented with 'cut and paste' with all the errors and mistakes that come with it. | A code generator avoids all mistakes from 'cut and paste' and implements all scaffolding code without any human intervention. |
| Sample reference application is not maintained and updated when new experience is gained. The sample cannot cover all variations. | Since the tool is used all the time it is motivated to continuously improve it. Knowledge is captured in the tool and other projects can make use of it. |
| The ideal structure might require some additional coding compared the quickest path for the moment. It is easy to take shortcuts when pressure on delivery is high. E.g. injecting entity manager and fire a query from the presentation layer instead of using a Repository to access the aggregate roots of the domain model. | Sculptor makes it easy to implement the correct design. The quickest solution is also the correct way of doing it from an overall design perspective. |
| Change in technology and new insights in better implementation is hard to introduce in existing applications. | Sculptor doesn't aim at solving this issue, but in practice it turns out that the technical infrastructure and glue code is typically generated and that means that it can easily be replaced by a new generator implementation. E.g. moving from Hibernate XML mapping files to JPA annotations is a major change that was a pretty straightforward migration when using Sculptor. |


We suggest automating some pieces of the development by using a code generator tool, such as Sculptor, to enforce consistency in the architecture.

## Sculptor Overview

You express your design intent in a textual specification, from which Sculptor generates high quality Java code and configuration. It is not a one time shot. The application can be developed incrementally with an efficient round trip loop. The generator is part of the build process (maven). So, as soon as you compile, build and package your application, sculptor generates all scaffold code. ![](/images/2010-03-02-promote-quality-with-sculptor/sculptor_overview3.png)

Sculptor is useful when developing typical enterprise or web applications that benefit from a rich and persistent domain model. Sculptor also provides a sophisticated CRUD GUI for administrative sections of the application or to serve as scaffolding for manually managed pages.

The generated code is based on well-known frameworks, such as JPA, Hibernate, Spring Framework, Spring Web Flow, JSF, Eclipse Rich Client Platform (RCP), and Java EE.

It is easy to remove the tool if it doesn't add value any more.

## Agile

An agile spirit in your project helps you deliver and focus on the right areas, hence improving the quality of what you deliver. Sculptor fits perfectly in agile projects. It has a short 'turn around' time when your model changes, i.e. when requirements change or you discover that you have misunderstood them.

Sculptor has very neat tools for communicating the model with stakeholders. The model itself is very readable, but if the person prefers a uml diagram can be generated. The generated CRUD GUI is also a good tool for discussing the domain with the customer. It is easier to talk about concrete running examples than abstract class diagrams.

The time from idea to a running prototype that can be discussed with the customer is important in an agile environment. Within 15 minutes you can go from scratch to a running application, including build scripts, Eclipse projects, domain model, JPA persistence, services, Web Flow application and much more. Thereafter you can continue by evolving the design, add manual code and regenerate.

## Customization

When selecting code generation tool it is crucial that you can easily adapt the tool to fit the requirements and frameworks you are working with. Maximum flexibility is achieved if you have the possibility to develop your own code generator, but that is a big investment. It is more efficient to use Sculptor, or develop your generator based on Sculptor.

There are massive amounts of technical decisions to be made for the reference architecture. Sculptor has many built in customization options to pick and choose from.

Sculptor also has good possibilities for extensibility if you prefer a design or framework that is not supported out-of-the-box. The generated result can be modified to meet your needs. Sculptor is using openArchitectureWare code generation platform, which makes it possible to change the code generation templates using Aspect-Oriented Programming (AOP) features. You can overwrite the definitions in the original templates and you can add your own templates to generate completely new artifacts. It is also possible to use AOP to exclude generation. For some special cases the default generation might not be appropriate and it is desirable to handle the special case with manual code instead. For example, assume we have a complex domain object and we need to do the JPA mapping manually.

Originally the generated implementation is using Spring with optional EJBs, but one customer decided to use pure EJB without Spring. We implemented this customization without problems and it is now contributed to Sculptor.

## Building Blocks of the Reference Architecture

It is easiest to base the reference architecture on well known design patterns. Much documentation comes for free and it effectively stops endless discussions.

Sculptor doesn't mandate how to design your system, but it provides a few building blocks, which are normally useful for normal enterprise or web applications. You are maybe already acquainted with these building blocks, since most of them has its origin in the books Domain-Driven Design and Patterns of Enterprise Architecture.

Below are brief descriptions of some of the building blocks and a teaser of what Sculptor provides.

### Entity and Value Objects

The domain model is defined using Entities and Value Objects. Entities have an identity and the state of the object may change during the life cycle. For Value Objects the values of the attributes are interesting, and not which object it is. Value Objects are typically immutable.

A sample of how to define a few domain objects in Sculptor Domain Specific Language (DSL):

![](/images/2010-03-02-promote-quality-with-sculptor/DDDSample_Cargo2_0.png)

Sculptor generates:

  * Data part of the domain objects.
  * JPA and Hibernate annotations for the attributes and associations of the domain model.
  * Constructors and accessor methods that reflect the changeability of the object.
  * equals, hashCode and toString
  * Database definition script.
  * Visualization class diagram.

Domain objects can of course also contain behavior; otherwise it wouldn't be a rich domain model. Behavior logic is always written manually, except for simple validation that is supported by adding Hibernate Validator annotations to the domain objects. Separation of generated and manually written code is done by a generated base class and manually written subclass. It is in the subclass you add methods to implement the behavior of the Domain Object.

### Aggregate

An Aggregate is a group of associated objects which are considered as one unit with regard to data changes. Each Aggregate has one root. The root is an Entity, and it is the only object accessible from outside. The root can hold references to any of the aggregate objects, and the other objects can hold references to each other, but an outside object can hold references only to the root object.

Sculptor will validate the reference constraints described above.

### Repository

The Repositories encapsulate all technical details for retrieving Domain Objects from the database. They are also used to make new objects persistent and to delete objects. The interface of the Repository speaks the Ubiquitous Language of the domain. It provides domain centric data access of the Aggregate roots of the domain model.

Example of a Repository, defined in Sculptor DSL:

![](/images/2010-03-02-promote-quality-with-sculptor/DDDSample_CargoRepository_0.png)

Sculptor generates the Repository boilerplate code and you fill in the logic and data access implementation manually. It also provides generic operations such as findById, findAll, findByQuery, findByKeys, save and delete. It is important that the API of the Repository communicates supported operations for the Aggregate root and therefore these generic operations are not added automatically to the Repository. They must be specified in the DSL, but that is very simple. It is possible to mark a Domain Object with scaffold to automatically generate some predefined CRUD operations in the Repository and corresponding Service.

### Service

Services act as a Service Layer around the domain model. It provides a well defined interface with a set of available operations to the clients. The transaction and error handling boundary is at the service layer.

A Service definition may look like this:

![](/images/2010-03-02-promote-quality-with-sculptor/DDDSample_Service.png)

Sculptor generates interface and implementation of the Service. You may add hand written code or use the automatic delegation mechanism to Repositories and other Services.

The choices for the service implementation may act as illustration of Sculptor's great customization options. Choose from:

  * POJO with implementation class and interface. Using Spring for dependency injection, error handling interceptor, and transactions.
  * EJB3 Stateless Session Bean delegating to POJO implementation. Using Spring for dependency injection and error handling interceptor, but container managed transactions.
  * Pure EJB3 Stateless Session Bean, without any Spring usage.

## Conclusions

For a development organization there is a lot of benefits of streamlining the number of design choices and agree on a reference architecture, including variations depending on system characteristics. Use Sculptor to promote this architecture and make sure it is implemented in a consistent way. Sculptor can likely be extended or customized to fit your choice of reference architecture.

The result of using Sculptor is better quality and also development productivity boost.

**Resources**

This article was originally published in [JayView](https://www.jayway.se/jayview.html) paper magazine
