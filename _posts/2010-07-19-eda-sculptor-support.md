---
layout: post
title: "EDA sculptor support"
description: ""
category: 
tags: [Sculptor,EDA,Scalability]
author: Andreas Källberg
navbar_name: blog
---
{% include JB/setup %}

In the previous [post][1] we gave an overview of [Event Driven Architecture][2]. It is a very big area, and you can use it to a lot of things.
It is a very good complement to DDD, and is a corner stone when building scalable systems.
When constructing systems it is a very nice match to accomplish loosely coupled modules and bounded contexts, i.e. business components. And much more.

We strive for simplicity, but at the same time not restricting us.

So, how can you use with Sculptor?
In the [1.9.0 release][8], we have focused on support for [Publish/Subscribe][3], [Command-Query Responsibility Segregation (CQRS)][4] and [Event Sourcing][5].

The most useful part is of course the pub/sub support. Event sourcing is an architectural style that has its niche. CQRS is also an architectural style that there has been some publicity around lately. CQRS uses pub/sub and can with advantage be constructed to use Event Sourcing.

To support the above, there are three central parts in our implementation:

  * An event bus abstraction
  * DomainEvent
  * CommandEvent


**"The bus"**

The event bus is an extremely simple API, with three different implementations ([in 1.9.0][8]), "Simple", [Spring Integration][6] and [Apache Camel][7].

The idea is that the central parts, i.e. pub/sub, should be easy to use. The only thing you have to work with is an event bus where you publish and subscribes to and from events. And if you need some non functional behavior (asynchronism, over the wire, etc) for your events, you plug in an event bus that can handle this requirements.
And as stated above, the easiest way to accomplish that with the [1.9.0 release][8] is to use Spring Integration or Apache Camel. But you can also choose to implement your own event bus.

You can publish and subscribe to and from the bus either declarative through the DSL, or programatically through the event bus API.


**DomainEvent vs CommandEvent**

`CommandEvent` is an instruction for something to happen. The system processes a `CommandEvent` and takes appropriate actions.

`DomainEvent` states fact - that something has happen. This fact is published to the rest of the world and the publisher just lets it go with no further interest in what happens to the event, i.e. who receives it and what they do.

If you compare it to an application API, `CommandEvent` could be the input API, while the `DomainEvent` is the output API for the application. I.e. the `CommandEvent` is what you can make the application perform, while the `DomainEvent` is what the application reports has happen.

As you probably figured out, `CommandEvent` is for implementing CQRS and Event Sourcing.

Finally, let us take quick look at how the notion for DomainEvents look like in the DSL.

~~~
DomainEvent ShipHasArrived {
    - ShipId ship
    - UnLocode port
}

DomainEvent ShipHasDepartured {
    - ShipId ship
    - UnLocode port
}
~~~

As you can see, you define a `DomainEvent` in the same way as for entities or value objects.

And to declare pub/sub:

~~~
Service TrackingService {
    @ShipHasArrived recordArrival(DateTime occurred, @Ship ship, @Port port)
                publish to shippingChannel;
}

Service Statistics {
    subscribe to shippingChannel
    int getShipsInPort(@UnLocode port);
    reset;
}
~~~

That was all for today, next time will it be more concrete with examples of how to use it.


   [1]: /2010/07/15/eda-overview
   [2]: https://en.wikipedia.org/wiki/Event-driven_architecture
   [3]: https://en.wikipedia.org/wiki/Publish/subscribe
   [4]: https://www.udidahan.com/2009/12/09/clarified-cqrs/
   [5]: https://martinfowler.com/eaaDev/EventSourcing.html
   [6]: https://www.springsource.org/spring-integration
   [7]: https://camel.apache.org/
   [8]: /documentation/whats-new#version-190
