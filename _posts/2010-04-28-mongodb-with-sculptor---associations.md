---
layout: post
title: "MongoDB with Sculptor - Associations"
description: ""
category: 
tags: [MongoDB,Sculptor,NoSQL,DDD]
author: Patrik Nordwall
navbar_name: blog
---
{% include JB/setup %}

This post is part of a [series of articles][1] describing the support for MongoDB in Sculptor. This post explains how associations and inheritance are managed.

MongoDB is not a relational database and there is no such thing as joins. You can still use associations between domain objects. Associations can be of two main categories, either embedded or reference by id.


### Aggregates

Aggregates are one of the core building blocks in Domain-Driven Design (DDD). MongoDB has perfect support for aggregates, since an associated object can be stored as an embedded document, i.e. it belongs to parent object and cannot be shared between several objects.

Let us repeat what DDD says about aggregates:

> An Aggregate is a group of associated objects which are considered as one unit with regard to data changes. The Aggregate is demarcated by a boundary which separates the objects inside from those outside. Each Aggregate has one root. The root is an Entity, and it is the only object accessible from outside. The root can hold references to any of the aggregate objects, and the other objects can hold references to each other, but an outside object can hold references only to the root object.
<small>_from [DDD Quickly][3]_</small>

Sculptor will validate the reference constraints described in the quote above. Repositories are only available for aggregate roots. Aggregates are defined with belongsTo or not aggregateRoot in the owned DomainObjects.

A typical aggregate in the [blog sample][4] is that `Comment` belongs to `BlogPost`

~~~
Entity BlogPost {
    String slug key
    String title
    String body
    DateTime published nullable
    - List comments opposite forPost
}

ValueObject Comment {
    not aggregateRoot
    - BlogPost forPost opposite comments
    String title
    String body
}
~~~

An aggregate can of course include several classes as in this sample:

~~~
Entity Cargo {
    - TrackingId trackingId key
    - Location origin required
    - Location destination required
    - Itinerary itinerary nullable opposite cargo
    - Set events opposite cargo
}

BasicType TrackingId {
    String identifier key
}

ValueObject Itinerary {
    belongsTo Cargo
    - Cargo cargo nullable opposite itinerary
    - List legs
}

ValueObject Leg {
    belongsTo Cargo
    - CarrierMovement carrierMovement
    - Location from
    - Location to
}
~~~

In above sample the `TrackingId`, `Itinary` and `Leg` are all stored toghether with the `Cargo`. [BasicTypes][2] are also stored as embedded documents.


### Reference by Id

The other alternative is to store ids of the referred objects. In the domain objects there are generated getters that lazily fetch associated objects from the ids. This means that you don't have to work with the ids yourself, you can follow associations as usual, but be aware that an invocation of such a getter might need to query the database.


~~~ java
Set writers = blog.getWriters();
~~~

In the same way you can modify unowned associations by working with objects rather than ids.

~~~ java
Author pn = new Author("Patrik");
pn = authorService.save(getServiceContext(), pn);
blog.addWriter(pn);
Author ak = new Author("Andreas");
ak = authorService.save(getServiceContext(), ak);
blog.addWriter(ak);
blogService.save(getServiceContext(), blog);
~~~

Referential integrity of the stored ids are not enforced. It must be handled by your program. Lazy getters of associations will not fail if referred to object is missing, they will return null for single value references and ignore missing objects for collection references. This means that you can cleanup dangling references by fetching objects, populate associations by invoking the getters and then save the object.


### Inheritance

The MongoDB feature of Sculptor has full support for inheritance. It is even possible to do polymorphic queries.

~~~
abstract Entity Media {
    String title !changeable

    Repository MediaRepository {
        List findByTitle(String title);
        protected findByCondition;
    }
}

Entity Book extends @Media {
    String isbn key length="20"
}

Entity Movie extends @Media {
    String urlIMDB key
    Integer playLength
    - @Genre category nullable
}
~~~

   [1]: /2010/04/27/mongodb-with-sculptor---introduction
   [2]: /2009/08/20/introducing-type
   [3]: http://www.infoq.com/minibooks/domain-driven-design-quickly
   [4]: https://github.com/sculptor/sculptor/tree/develop/sculptor-examples/mongodb-samples/blog-mongodb
