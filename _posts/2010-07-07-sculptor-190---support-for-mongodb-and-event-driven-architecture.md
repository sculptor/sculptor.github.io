---
layout: post
title: "Sculptor 1.9.0 - Support for MongoDB and Event Driven Architecture"
description: ""
category: 
tags: [Sculptor, Release]
author: Sculptor Team
navbar_name: blog
---
{% include JB/setup %}

The Sculptor development team has a track record delivering around 3 releases per year. We have delivered 10 releases in total. That means that the core pieces are rock solid. I have personally used it successfully together with 15 other developers on daily basis for about a year now. It simply works very well.

We care about bugfixing and making small improvements. At the same time we are excited about learning new technology and using emergent design. This release contains two new features in the area of scalability. Persistence backed with MongoDB and support for Event-Driven Architecture.

**MongoDB** bridges the gap between key-value stores (which are fast and highly scalable) and traditional RDBMS systems (which provide rich queries and deep functionality). I think this makes MongoDB very interesting for applications that need high-performance and/or scalability, but also prefer using a rich persistent domain model with complex associations. The schema less structure is attractive from a developer productivity perspective, which is one of the two goals with Sculptor (quality is the other).

Sculptor generates data mapper classes that converts domain objects to/from MongoDB data structures, DBObjects. This makes it easy to use a domain model à la DDD with automatic mapping to MongoDB data structures.

Sculptor provides generic repository operations for use with MongoDB. This includes operations such as save, delete, findById, findByKey, findByCondition, and some more. You get CRUD operations, and GUI, for free.

Queries can be expressed with a slick fluent api that support code completion and refactoring.

Rich support for associations. Aggregates are stored as a embedded documents. Other associations are stored with referring ids. In the domain objects there are generated getters that lazily fetch associated objects from the ids. This means that you don't have to work with the ids yourself, you can follow associations as usual.

Read more about how to use MongoDB with Sculptor [here][1].


**Event-Driven Architecture (EDA)** is a good complement to Domain-Driven Design. We think EDA is an important ingredient for building scalable systems. It is also an enabler for designing loosely coupled modules and bounded contexts.

For this Sculptor makes it possible to define Domain Events in the model in similar way as Entities and Value Objects. Sculptor also provide a mechanism to publish and subscribe through a simple event bus. This is done either in a declarative way in the model, or programatically with a simple API.

Implementations of the event bus that integrates with Apache Camel and Spring Integration are available out-of-the-box.

In addition to publish/subscribe Sculptor also has support for CQRS and EventSourcing.

CQRS is about separating commands (that change the data) from the queries (that read the data). Separate subsystems take care of answering queries (reporting) and the domain for processing and storing updates can stay focused.

Event Sourcing makes it possible to see how we got to the current state and query how the state looked liked in the past. Essentially it means that we have to capture all changes to an application state as a sequence of events. Sculptor provides a default implementation of EventSourcing, which can be extended and customized to fit specific needs.

Read more about how to use the new event features of Sculptor [here][2]. We will blog more about this topic later, so stay tuned.

   [1]: /documentation/mongodb-tutorial
   [2]: /documentation/event-driven-tutorial