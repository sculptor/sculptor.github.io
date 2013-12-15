---
layout: post
title: "MongoDB with Sculptor - Repository"
description: ""
category: 
tags: [MongoDB,Sculptor,NoSQL,DDD]
author: Patrik Nordwall
navbar_name: blog
---
{% include JB/setup %}

This post is part of a [series of articles][1] describing the support for MongoDB in Sculptor. This post explains how [Repositories][2] with useful operations are defined and how queries can be expressed in terms of the domain.

In the visualization of the [blog sample][1] you maybe noticed that there are Services with `findById`, `findAll`, `save` and `delete` even though they were not explicitly defined in the model. They come from that the domain objects are marked with with `scaffold`. That automatically generates some predefined CRUD operations in the Repository and corresponding Service.

~~~
Entity BlogPost {
    scaffold
    - Blog inBlog
    String slug key
    String title
    String body
    DateTime published nullable
    Set tags
    - Author writtenBy
    - List comments opposite forPost
}
~~~

You can also define the needed operations explicitly like this:

~~~
Entity BlogPost {
    - Blog inBlog
    String slug key
    String title
    String body
    DateTime published nullable
    Set tags
    - Author writtenBy
    - List comments opposite forPost

    Repository BlogPostRepository {
        save;
        delete;
        findAll(PagingParameter pagingParameter);
        findById;
        findByKey;
        List findPostsInBlog(@Blog blog);
        List findPostsWithComments => AccessObject;
        List findPostsWithTags(Set tags);
        protected findByCondition;
    }
}
~~~

You don't need to do any manual coding for the built in operations, such as `save`, `delete`, `findAll`, `findById`, `findByKey`.
`findByKey` is for the natural key, i.e. the attributes marked with key (slug above).

There is good support for pagination and sorting.

As you see in the above sample it is also possible to define your own repository operations, such as `findPostsWithTags`.


### findByCondition

The most interesting finder is `findByCondition`. Queries can expressed with an internal DSL that support code completion and refactoring. It is best illustrated with a few samples.

~~~ java
public List findPostsInBlog(Blog blog) {
    List condition = criteriaFor(BlogPost.class)
        .withProperty(inBlog()).eq(blog)
        .orderBy(published()).descending()
        .build();
    return findByCondition(condition);
}

public List findPostsWithGreatComments() {
    List condition = criteriaFor(BlogPost.class)
        .withProperty(comments().title()).ignoreCaseLike(".*great.*")
        .and().withProperty(published()).isNotNull()
        .orderBy(published()).descending().build();
    return findByCondition(condition);
}
~~~

The `ConditionalCriteriaBuilder` has support for defining queries with `eq`, `like`, `between`, `lessThan`, `greaterThan`, `in`, `and`, `not`, `orderBy`, and some more.

You can of course also work directly with the underlaying MongoDB `DBCollection` and `DBObject` for doing advanced queries that might not be supported by `findByCondition`.

   [1]: /2010/04/27/mongodb-with-sculptor---introduction
   [2]: /documentation/advanced-tutorial#repository
