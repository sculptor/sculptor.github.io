---
layout: post
title: "Support for mongoDB started"
description: ""
category: 
tags: [Sculptor,NoSQL,MongoDB]
author: Patrik Nordwall
navbar_name: blog
---
{% include JB/setup %}

I think NoSQL datastores are a very interesting complement to relational databases. I think [mongoDB][1] is a good choice for the rich persistent domain models that are typical for a Sculptor application. Today's [article][2] at InfoQ about mongoDB mention the relation between document stores and DDD.

We have started prototyping and it looks promising. The idea is that that Sculptor will provide:

  * Access objects (implementation of repository operations) that interact with mongoDB.
  * Good query support (e.g. [findByCondition][3]).
  * Automatic mapping between domain objects and mongoDB documents (DBObject) will be generated.
  * Different types of association will be supported. Aggregates are typically embedded documents and other associations are stored as id references. Maybe we will provide a mechanism to lazy load associated documents.

   [1]: http://www.mongodb.org/
   [2]: http://www.infoq.com/news/2010/03/mongodb
   [3]: /documentation/advanced-tutorial#findbycondition
