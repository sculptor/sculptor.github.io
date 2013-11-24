---
layout: post
title: "Event Sourcing with Sculptor"
description: ""
category: 
tags: [EDA,CQRS,Sculptor]
author: Patrik Nordwall
navbar_name: blog
---
{% include JB/setup %}

A while a go Greg Young published the [Super Simple CQRS Example][1]. There are also some good documents of how to use events as storage mechanism at the [CQRS Info Site][2].

I have implemented the same example with Java and Sculptor. In this post I will describe how the Event Sourcing is implemented. Even though my implementation is facilitated by using Sculptor and MongoDB the design can easily be done with other techniques, such as JPA or plain JDBC, and without Sculptor.

A domain model typically holds current state of the world. Event Sourcing makes it possible to see how we got to this state. Essentially it means that we have to capture all changes to an application state as a sequence of events. These events can be used to reconstruct current and past states.

The sample application is a simplified inventory of items. In line with CQRS it has a strict separation of commands and queries. Changes to the `InventoryItem` domain object are published as Domain Events. The `InventoryItem` can be modified with a few commands that I have represented as operations in a Service.

~~~ java
Service InventoryFacade {
    inject @InventoryItemRepository
    createInventoryItem(String itemId, String name);
    deactivateInventoryItem(String itemId);
    renameInventoryItem(String itemId, String newName);
    checkInItemsToInventory(String itemId, int count);
    removeItemsFromInventory(String itemId, int count);
}
~~~

All these commands are handled in the same way, except `createInventoryItem`, which is slightly different. The Service looks up the Entity and calls corresponding method on the domain object.

~~~ java
public void deactivateInventoryItem(String itemId) {
    InventoryItem item = tryGetItem(itemId);
    item.deactivate();
    getInventoryItemRepository().save(item);
}
~~~

The Entity performs validation and creates Domain Event for state changes.

~~~ java
public void deactivate() {
    if (!isActivated())
        throw new IllegalStateException("already deactivated");
    applyChange(new InventoryItemDeactivated(new Date(), getItemId()));
}
~~~

In this case `InventoryItem` has a flag (state) indicating that the item is activated or not. Note that this state is not changed directly, but it is changed later as a result of the `InventoryItemDeactivated` event. All changes are done with Domain Events, which is important, because we don't save current state, we only save the Domain Events. The activated flag is not stored explicitly.

The `Domain Event` is applied and added to a list of changes, which later will be stored and published.

~~~ java
private void applyChange(InventoryItemEvent event) {
    applyChange(event, true);
}

private void applyChange(InventoryItemEvent event, boolean isNew) {
    DynamicMethodDispatcher.dispatch(this, event, "apply");
    if (isNew) {
        changes.add(event);
    } else {
        setVersion(event.getAggregateVersion());
    }
}

public void apply(InventoryItemDeactivated event) {
    setActivated(false);
}
~~~

When saving, the `InventoryItem` instance is saved, but only as a key and version. The version is used for detecting concurrent modifications (optimistic locking). Additionally, the changes, the Domain Events are stored. The version number of `InventoryItem` is stored in Each Domain Event. An additional sequence number is also used to ensure the correct order of the events.

We need to override the default save method in the Repository to handle these sequence numbers and also save the events.

~~~ java
@Override
public InventoryItem save(InventoryItem entity) {
    InventoryItem saved = super.save(entity);

    List changes = entity.getUncommittedChanges();
    changes = applyVersionToChanges(changes, saved.getVersion());
    for (InventoryItemEvent each : changes) {
        getInventoryItemEventRepository().save(each);
    }
    entity.markChangesAsCommitted();

    return saved;
}

private List applyVersionToChanges(
    List changes, long version) {
    List result = new ArrayList();
    long sequence = version * 1000;
    for (InventoryItemEvent each : changes) {
        result.add(each.withAggregateVersion(version).withChangeSequence(sequence));
        sequence++;
    }
    return result;
}
~~~

When saving the Domain Events they are also published to a topic, which the read side subscribes on. That is handled with the publish/subscribe mechanism in Sculptor. In the model we simply need to specifiy publish on the save method.

~~~
abstract DomainEvent InventoryItemEvent {
    persistent
    String itemId index
    Long aggregateVersion nullable
    Long changeSequence nullable

    Repository InventoryItemEventRepository {
        save publish to inventoryItemTopic;
        List findAllForItem(String itemId);
        protected findByCondition;
    }
}

DomainEvent InventoryItemDeactivated extends @InventoryItemEvent {
}
~~~

In the read side we add subscribers to this topic.

~~~
Service InventoryListView {
    subscribe to inventoryItemTopic
    inject @InventoryItemListRepository
}

Service InventoryItemDetailView {
    subscribe to inventoryItemTopic
    inject @InventoryItemDetailsRepository
}
~~~

Alright, then we are almost done. One more thing though, when retrieving a `InventoryItem` we must replay all events to recreate current state. We do that by overriding the default `findByKey` method in the Repository.

~~~ java
@Override
public InventoryItem findByKey(String itemId) throws InventoryItemNotFoundException {
    InventoryItem result = super.findByKey(itemId);

    loadFromHistory(result);

    return result;
}

private void loadFromHistory(InventoryItem entity) {
    List history = getInventoryItemEventRepository().findAllForItem(entity.getItemId());
    entity.loadFromHistory(history);
}
~~~

To retrieve all events we use a simple query in the `InventoryItemEventRepository`.

~~~ java
public List findAllForItem(String itemId) {
    List criteria = criteriaFor(InventoryItemEvent.class).withProperty(itemId()).eq(itemId).orderBy(changeSequence()).build();
    return findByCondition(criteria);
}
~~~

The loaded events are applied to the `InventoryItem` Domain Object.

~~~ java
public void loadFromHistory(List history) {
    for (InventoryItemEvent each : history) {
        applyChange(each, false);
    }
}

private void applyChange(InventoryItemEvent event, boolean isNew) {
    DynamicMethodDispatcher.dispatch(this, event, "apply");
    if (isNew) {
        changes.add(event);
    } else {
        setVersion(event.getAggregateVersion());
    }
}

public void apply(InventoryItemCreated event) {
    setActivated(true);
}

public void apply(InventoryItemDeactivated event) {
    setActivated(false);
}

public void apply(Object other) {
    // ignore
}
~~~

In this example we have used a naive approach when loading the `InventoryItem` by replaying all historical events. For Entities with a long life cycle it can be too many events. Then we can use a snapshot technique, which I will describe in a [separate blog post][3].

The complete source code for this example can be found here: [https://github.com/patriknw/sculptor-simplecqrs/tree/event_sourcing_without_snapshots][4]

   [1]: http://github.com/gregoryyoung/m-r
   [2]: http://cqrs.wordpress.com/documents/
   [3]: /2010/10/29/event-sourcing-with-sculptor---snapshots
   [4]: https://github.com/patriknw/sculptor-simplecqrs/tree/event_sourcing_without_snapshots
