---
layout: post
title: "Sculptor 2.1.0 Released"
description: ""
category: 
tags: [Sculptor, Release]
author: Sculptor Team
navbar_name: blog
---
{% include JB/setup %}

One very useful new feature is the generated finder methods. It is now possible to specify the query in the model file and let sculptor generate the code for the repository operations. In some cases the query can be derived from the method signature only.

This will generate a query returning all persons with the specified first name.

~~~ java
Entity Person {
	String firstName
	String lastName
	Date birthDate

	PersonRepository {
		findBy(String firstName);
	}
}
~~~

More advanced conditions can also be specified

~~~ java
PersonRepository {
	findBornBefore(Date bd) condition="birthDate < :bd" orderBy="firstName";
	findByName(String name) condition="lastName i= :name or firstName i= :name";
}
~~~

Sculptor generates `findByCondition` expressions and will report at generation time if there are any errors in the queries.

Another nice addition is the generated builder classes that provides a fluent interface to build domain objects in a manner that can be easier to work with and read.

~~~ java
Book book = book()
	.createdBy("me")
	.createdDate(now)
	.title("Ender's Game")
	.isbn("Some-ISBN")
	.addMediaCharacter(mediaCharacter()
		.name("Ender")
		.build())
	.build();
~~~

[Read more about the release][1].

   [1]: http://fornax.itemis.de/confluence/display/fornax/0.%2BWhat%27s%2BNew%2B%28CSC%29#0.What%27sNew%28CSC%29-Version2.1.x
