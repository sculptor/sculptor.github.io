---
layout: post
title: "MongoDB with Sculptor - Data Mapper"
description: ""
category: 
tags: [Sculptor,NoSQL,MongoDB,DDD]
author: Patrik Nordwall
navbar_name: blog
---
{% include JB/setup %}

This post is part of a [series of articles][1] describing the support for MongoDB in Sculptor. This post explains how the mapping between domain objects and MongoDB data objects is performed.

When working with mongoDB Java API the [DBObject][2] plays a central role. It is like a key value Map. Values can be of most types and also collections and other DBObjects for nested documents. Sculptor also adds support for Enums and Joda date time classes.

Implementing mapping of domain objects to DBObjects is a tedious and error prone programming task that is a perfect candidate for automation. Sculptor comes in handy for this. Sculptor generates data mapper classes that converts domain objects to DBObjects. It is useful to not have to write those mappers by hand.

An early definition of a BlogPost in the blog sample might look like this in Sculptor model:

~~~
Entity BlogPost {
    String slug key
    String title
    String body
}
~~~

From this Sculptor generates domain object class in java with appropriate getters, setters, constructors, equals, etc. Sculptor also generates a mapper for converting the BlogPost to/from DBObject. The generated code looks like this:

~~~ java
public DBObject toData(BlogPost from) {
    if (from == null) {
        return null;
    }

    DBObject result = new BasicDBObject();

    if (from.getId() != null) {
        ObjectId objectId = ObjectId.massageToObjectId(from.getId());
        result.put("_id", objectId);
    }

    result.put("slug", from.getSlug());
    result.put("title", from.getTitle());
    result.put("body", from.getBody());

    return result;
}
~~~

and in the other direction:

~~~ java
public BlogPost toDomain(DBObject from) {
    if (from == null) {
        return null;
    }

    String slug = (String) from.get("slug");

    BlogPost result = new BlogPost(slug);

    if (from.containsField("_id")) {
        ObjectId objectId = (ObjectId) from.get("_id");
        String idString = objectId.toStringMongod();
        IdReflectionUtil.internalSetId(result, idString);
    }

    if (from.containsField("title")) {
        result.setTitle((String) from.get("title"));
    }
    if (from.containsField("body")) {
        result.setBody((String) from.get("body"));
    }

    return result;
}
~~~

I'm happy that I don't have to write and especially maintain that kind of code. The generation is not a one time shot. When you change the model the domain objects and the mappers are regenerated.

Domain objects can of course also have behavior, otherwise it wouldn't be a rich domain model. The behavior logic is always written manually. Separation of generated and manually written code is done by a generated base class and manually written subclass, a gap class. It is in the subclass you add methods to implement the behavior of the domain object. The subclass is also generated, but only once, it will never be overwritten by the generator.

You might need to add some hand written code to customize the mappers. That is easy. In the model you can specify that you need a subclass that you can implement yourself. This is very useful for doing data migration.

It can also be good to know that fields marked with transient are are not stored, but they are loaded if they exist in the retrieved documents. This can also be used for data migration.

By default the names in the data store are the same as the names in the Java domain objects. In case you need to use other names it is possible to define that in model with `databaseTable` and `databaseColumn`.

~~~
Entity BlogPost {
    databaseTable="BlogEntries"
    String slug key
    String title
    String body databaseColumn="content"
}
~~~

You have probably selected MongoDB for its low latency. Then you want the data mapper to be fast also. The mappers provided by Sculptor are generated code that runs at full speed. Alternative solutions using reflection or intermediate String JSON format is probably not as fast.

   [1]: /2010/04/27/mongodb-with-sculptor---introduction
   [2]: http://api.mongodb.org/java/2.11.3/com/mongodb/DBObject.html
