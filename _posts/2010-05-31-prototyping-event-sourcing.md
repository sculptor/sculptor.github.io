---
layout: post
title: "Prototyping Event Sourcing"
description: ""
category: 
tags: [MongoDB,Sculptor,EDA]
author: Patrik Nordwall
navbar_name: blog
---
{% include JB/setup %}

In Domain Language Newsletter March 2010 we could read Eric Evans writeup on Domain Events.

> Over the last few years it has become clear that it is very useful to add a new pattern to the DDD "Building Blocks" (Entities, Value Objects, Services, etc.) -- Domain Events. This pattern has been around for a long time. Martin Fowler wrote [a good description][1].

Andreas and I have lately been prototyping on some mechanism to better support Event-Driven Architecture in Sculptor. In this article I will describe how to do [Event Sourcing][2].

A domain model typically holds current state of the world. Event Sourcing makes it possible to see how we got to this state and query how the state looked liked in the past. Essentially it means that we have to capture all changes to an application state as a sequence of events. These events can be used to reconstruct current and past states.

In the prototype we have used Martin Fowler's [shipping system example][2]. The full source code for the sample is available on [GitHub][3]. I have developed Event Sourcing with existing Sculptor mechanisms. Next step will be to move the general pieces into the tool so that it can be easily used. I would like to share the ideas right away in this blog.

The domain model for the shipping system is very simple. Ships that carry cargo and move between ports. It looks like this in Sculptor DSL:

~~~
Entity Ship {
    gap
    - ShipId shipId key
    String name
    - Port port nullable
    - Set cargos

    Repository ShipRepository {
        gap
        inject @CargoRepository
        save;
        findByKey;
    }
}

Entity Port {
    - UnLocode unlocode key;
    String city
    - Country country

    Repository PortRepository {
        save;
        findByKey;
    }
}

Entity Cargo {
    gap
    String cargoId key
    boolean hasBeenInCanada

    Repository CargoRepository {
        save;
        findByKey;
    }
}

BasicType ShipId {
    String identifier key
}

"United nations location code."
BasicType UnLocode {
    String identifier key
}

enum Country {
    US,
    CANADA
}
~~~

When a ship arrives at port it is registered in the `TrackingService`, `recordArrival` operation. Instead of taking that information and directly save it to current domain objects, the `TrackingService` creates a `DomainEvent` and pass it in to the processor for further execution.

~~~ java
public void recordArrival(DateTime occured, Ship ship, Port port) {
    DateTime now = new DateTime();
    ArrivalEvent event = new ArrivalEvent(occured, now, ship.getShipId(), port.getUnlocode());
    getDomainEventProcessor().process(event);
}
~~~

![][4]


The `DomainEventProcessor` stores the event, which is important for audit and possibility to replay events to reconstruct past states. The `DomainEventProcessor` is generic, knows nothing about the shipping domain. It dispatches the event to an `EventHandler` that knows how to process the shipping specific events. As a start I use simple Spring dependency injection to select `EventHandler`, but we will probably use a slightly more sophisticated mechanism later. The important thing is that the generic `DomainEventProcessor` doesn't know how to process events, so it delegates to application specific event handlers.

A useful utility for dispatching events to separate methods is available in [commons beanutils][10]. The `ShippingEventHandler` looks like this (error handling removed):

~~~ java
public void handleEvent(DomainEvent event) {
    dispatch(event);
}

/**
 * Runtime dispatch to handle method with correct event parameter type
 */
protected void dispatch(DomainEvent event) {
    MethodUtils.invokeMethod(this, "handle", new Object[] { event });
}

public void handle(ArrivalEvent event) {
    Ship ship = getShipRepository().findByKey(event.getShip());
    Port port = getPortRepository().findByKey(event.getPort());
    ship.arrival(port);
    getShipRepository().save(ship);
}

public void handle(DepartureEvent event) {
    Ship ship = getShipRepository().findByKey(event.getShip());
    Port port = getPortRepository().findByKey(event.getPort());
    ship.departure(port);
    getShipRepository().save(ship);
}
~~~

The events are also defined in Sculptor DSL. So far I have used ordinary ValueObjects for the events but we will add a special syntax for defining events like this:

~~~
DomainEvent ArrivalEvent {
    - ShipId ship
    - UnLocode port
}

DomainEvent DepartureEvent {
    - ShipId ship
    - UnLocode port
}
~~~

What have we done? We have made the design more complicated by an intermediate event step, but we have also a foundation for all the exciting things that can be done with Event Sourcing.

We can query the state of a ship for a specific point in time, and it is not only the state of the ship, the complete domain model can be used as usual at a specific point in time.

~~~ java
DateTime to = travelStart.plusDays(15);
ReplaySpecification replaySpec = new ReplaySpecification().withTo(to).withTarget(tmpDb);
domainEventProcessor.replay(replaySpec);
DbManager.setThreadInstance(tmpDb);
Ship ship = referenceDataService.getShip(shipId);
Port port = ship.getPort();
Set cargos = ship.getCargos();
~~~

When doing this prototype we have used [MongoDB][9], which is excellent for these kind of features because:

  * it is schema-less, so any event type can be stored without predefined schema
  * it has low latency, so replay of events are fast
  * it is easy to make new database instances and copy database, which make replay and snapshot mechanisms simple

In the above example we replay the events into a temporary database (tmpDb), a [Parallel Model][5]. This is a full featured MongoDB instance and therefore we have all the ordinary query capabilities, i.e. exactly the same domain model including repositories can be used with the temporary database as with the ordinary database.
{: .alert}

The generic `DomainEventProcessor` provides several methods to operate on the events. For example to speed up replay of large data volumes it is possible to create snapshots. Latest preceding snapshot is used as base when replaying, i.e. only the events after the snapshot time need to be processed.

The event processor module is defined like this in the prototype. The idea is that it will be provided automatically by Sculptor. Of course with customization possibilities to support variations.

~~~
Module event {
    Service DomainEventProcessor {
        inject @DomainEventRepository
        inject @SnapshotRepository
    
        process(@DomainEvent event);
        getAllEvents => DomainEventRepository.findAll;
        replayAll;
        replayUpTo(DateTime timePoint);
        replay(@ReplaySpecification spec);
        save(List events);
        String createSnapshot(DateTime timePoint);
    }

    abstract ValueObject DomainEvent {
        gap
        DateTime occured index;
        DateTime recorded;
    
        Repository DomainEventRepository {
            save;
            save(List entities);
            findAll(PagingParameter pagingParameter);
            findBetween(DateTime from, DateTime to, PagingParameter pagingParameter);
            protected findByCondition(PagingParameter pagingParameter);
        }
    }

    ValueObject ReplaySpecification {
        not persistent
        DateTime from nullable
        DateTime to nullable
        DB target nullable
    }

    ValueObject SnapshotInfo {
        DateTime timePoint index
        String name
    
        Repository SnapshotRepository {
             @SnapshotInfo findLatest(DateTime timePoint);
             String snapshotName(DateTime timePoint);
             @SnapshotInfo createSnapshot(DateTime timePoint);
             copyDb(String fromDbName, String toDbName) => CopyDbAccessObject;
             protected findByCondition;
             protected save;
        }
    }
}
~~~

We are also prototyping how to add other event mechanisms, such as publish/subscribe and integration with various products, such as [Spring Integration][6], [Apache Camel ][7]and [Akka][8]. More about that later.

   [1]: http://martinfowler.com/eaaDev/DomainEvent.html
   [2]: http://martinfowler.com/eaaDev/EventSourcing.html
   [3]: https://github.com/sculptor/sculptor/tree/develop/sculptor-examples/mongodb-samples/sculptor-shipping
   [4]: /images/2010-05-31-prototyping-event-sourcing/EventSourcingDesign.png
   [5]: http://martinfowler.com/eaaDev/ParallelModel.html
   [6]: http://www.springsource.org/spring-integration
   [7]: http://camel.apache.org/
   [8]: http://akkasource.org/
   [9]: http://mongodb.org/
   [10]: http://commons.apache.org/beanutils/
