---
layout: post
title: "Event Sourcing with Sculptor - Snapshots"
description: ""
category: 
tags: [EDA,CQRS,Sculptor,Scalability]
author: Patrik Nordwall
navbar_name: blog
---
{% include JB/setup %}

[Yesterday I described][1] how events can be used as storage mechanism. Instead of storing current state we store each change as a Domain Event. Current state is reconstructed by loading and replaying all historical events. For Entities with a long life cycle it can be too many events. This can be solved with an optimization that is based on periodically storing a rolling snapshot of current state.

Let us look at the code in the Sculptor port of the [Simple CQRS Example][2] to understand what this means in practice.

When loading InventoryItem we start by applying latest snapshot, if any, and thereafter replaying the events after the snapshot. This is the code in the Repository:

~~~ java
@Override
public InventoryItem findByKey(String itemId) throws InventoryItemNotFoundException {
    InventoryItem result = super.findByKey(itemId);

    loadFromHistory(result);

    return result;
}

private void loadFromHistory(InventoryItem entity) {
    InventoryItemSnapshot snapshot = getInventoryItemSnapshotRepository().getLatestSnapshot(entity.getItemId());
    entity.applySnapshot(snapshot);
    long snapshotVersion = snapshot == null ? 0 : snapshot.getVersion();

    List history = getInventoryItemEventRepository().findAllAfter(entity.getItemId(), snapshotVersion);
    entity.loadFromHistory(history);
}
~~~

That is how the snapshots are used when loading `InventoryItem`. Let us see how they are saved. We define the Snapshot Value Object for `InventoryItem` like this

~~~
ValueObject InventoryItemSnapshot {
    String itemId index
    boolean activated
    Long version

    Repository InventoryItemSnapshotRepository {
        @InventoryItemSnapshot getLatestSnapshot(String itemId);
        protected findByCondition(PagingParameter pagingParameter);
        save;
    }
}
~~~

Here we store the state as explicit attributes, which is simple with Sculptor, since we got the persistence (to MongoDB or JPA) for free, but it can also be stored as a blob (encoded as xml, protobuf or whatever you prefer). In this example the only state is the `activated` flag, but it can be much more in a real application.

The storage of the snapshot is triggered by a subscriber on the ordinary Domain Event flow. Simply defined as this in the model:

~~~
Service InventoryItemSnapshotter {
    subscribe to inventoryItemTopic
    inject @InventoryItemRepository
    inject @InventoryItemSnapshotRepository
}
~~~

The implementation calculates how many events has passed since previous snapshot by comparing version numbers. When the delta exceeds a threshold (e.g. 100 events) a snapshot is created and saved.

~~~ java
public void receive(Event event) {
    if (!(event instanceof InventoryItemEvent)) {
        return;
    }

    InventoryItemEvent inventoryItemEvent = (InventoryItemEvent) event;
    String itemId = inventoryItemEvent.getItemId();

    InventoryItemSnapshot snapshot = getInventoryItemSnapshotRepository().getLatestSnapshot(itemId);
    long snapshotVersion = snapshot == null ? 1 : snapshot.getVersion();
    long eventVersion = inventoryItemEvent.getAggregateVersion() == null ? 1 : inventoryItemEvent.getAggregateVersion();
    if (eventVersion - snapshotVersion >= VERSION_DELTA) {
        takeSnapshot(itemId);
    }
}

private void takeSnapshot(String itemId) {
    InventoryItem item;
    try {
        item = getInventoryItemRepository().findByKey(itemId);
    } catch (InventoryItemNotFoundException e) {
        log.warn("takeSnapshot failed: " + e.getMessage());
        return;
    }

    InventoryItemSnapshot snapshot = item.createSnapshot();
    getInventoryItemSnapshotRepository().save(snapshot);
}
~~~

By using snapshots we can dramatically improve performance for loading Entities with many historical changes. However, you can always start development without snapshotting and add it later, as a performance enhancement.

Also, note that snapshots and event are immutable and therefore we have great opportunities for using caching for improving performance.

The complete source code for this example is available here: [https://github.com/patriknw/sculptor-simplecqrs/][3]

   [1]: /2010/10/28/event-sourcing-with-sculptor
   [2]: http://github.com/gregoryyoung/m-r
   [3]: https://github.com/patriknw/sculptor-simplecqrs/
