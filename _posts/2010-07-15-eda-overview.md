---
layout: post
title: "EDA overview"
description: ""
category: 
tags: [Sculptor,EDA,Scalability]
author: Andreas Källberg
navbar_name: blog
---
{% include JB/setup %}

As stated in the [last post][1], we would like to talk a bit (very shortly) about EDA in a broader sense to give an overview of it and to give you a sense for what we read into the term EDA. This is quite important as a background when we go further and explain our interpretation and implementation of it.

[Event Driven Architecture][10] is a very broad area. But in a short sentence its about:

Applications and systems that produce, consume and reacts on events.

And even if there is a lot of attention on EDA as means of implementing integration between applications and systems, EDA can be applied within applications and even in parts of applications.

The benefits of an EDA is:

  * loosely coupled systems (or internals of a system)
  * high performance (fire and forget)
  * high scalability

Of course this comes with a trade-off. An extra abstraction is added, i.e. it becomes more complex.

In it's simplest form, EDA is about the [Observer Pattern][2]. Something happens somewhere, and 0 to n parties is interested in that happening. From this simple pattern, all interpretations, implementations and usages of EDA are spawn.

So, before we go into how we have implemented support for EDA in Sculptor, we will give you some examples of what shapes we think EDA can take. Big and small...here we go:

* **GUI's**

  Swing uses the Observer pattern. Many different GUI frameworks uses an event driven approach. Java Server Faces works with events, event handlers and event actions. Etc...

* **DomainEvent**

  A [Domain Event][3] is registration of something that has happen. It might be of interest to 0 to n consumers. The thing is; The producer doesn't care, it just tell the world that this thing has happen and happily goes on with it's life.

* **PubSub**

  Publisher and Subscribers is a central part of all event driven integrations. It is an implementation of the Observer pattern. It often comes with persistent events. Topics in the Java/JMS tech domain is an implementation of it.

* **SOA2.0**

  Now when SOA has been around for some years the next thing is SOA2.0. In the 2.0 version of SOA events plays a central role as a complement to services.


An event-driven system typically consists of event emitters (or agents) and event consumers (or sinks). Sinks have the responsibility of applying a reaction as soon as an event is presented. The reaction might or might not be completely provided by the sink itself. For instance, the sink might just have the responsibility to filter, transform and forward the event to another component or it might provide a self contained reaction to such event. The first category of sinks can be based upon traditional components such as message oriented middleware while the second category of sinks (self contained online reaction) might require a more appropriate transactional executive framework, for example an ESB.

* **Event servers**

  Event servers are servers that are specialized in processing events. They often provide filter/query capabilities for events. [WebLogic Event Server][4] is an example of this kind of product.

* **Event sourcing**

  All changes to an application is recorded as a series of events. An applications current state can be queried. But not only that, an application state can be rolled forward or backwards as you wish, giving you a lot of power. [Martin Fowler][5] has written all about it [here][6]. Also, Patrik has written a couple of blog entries of how he has played with it and now also added support for it in Sculptor, see [here][7] and [here][8].

* **CQRS**

  Command and Query Responsibility Segregation is a very cool area. I will not try to explain it in detail, it is very well done [here][9].

  Simplified it is about separating commands (that change the data) from the queries (that read the data). Separate subsystems take care of answering queries (reporting) and the domain for processing and storing updates can stay focused. The result of the commands are published to query subsystems, each optimized for its purpose.


Ok, that was probably a very short and incomplete list of EDA related topics, but it gave you a taste for it and a ground for further reading. We will use it as a foundation when we talk about how we think about and implements EDA.

Next up, how we support EDA in Sculptor.

   [1]: /2010/07/12/eda-intro
   [2]: http://en.wikipedia.org/wiki/Observer_pattern
   [3]: http://martinfowler.com/eaaDev/DomainEvent.html
   [4]: http://docs.oracle.com/cd/E13213_01/wlevs/docs20/
   [5]: http://martinfowler.com/
   [6]: http://martinfowler.com/eaaDev/EventSourcing.html
   [7]: /2010/05/31/prototyping-event-sourcing
   [8]: /2010/06/01/event-sourcing-snapshots
   [9]: http://www.udidahan.com/2009/12/09/clarified-cqrs/
   [10]: http://en.wikipedia.org/wiki/Event-driven_architecture
