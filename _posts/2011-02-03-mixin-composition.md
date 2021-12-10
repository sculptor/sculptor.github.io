---
layout: post
title: "Mixin Composition"
description: ""
navbar_name: blog
category: 
tags: [DDD, OO, Scala, Sculptor]
author: Patrik Nordwall
---
{% include JB/setup %}

I think one of the best features in [Scala][1] is [traits][2]. Using traits it is possible to compose small pieces of behavior and state in an elegant way. I miss traits when I use Java. To mitigate that problem I have implemented support for traits in Sculptor. This article illustrates how this tool can be used for designing good, rich, domain models with traits.

Traits provide a mixin composition mechanism that is missing in Java. Similar to interfaces in Java, traits are used to define object types by specifying the signature of the supported methods. Unlike interfaces, traits can be partially implemented; i.e. it is possible to define implementations for some methods. Similar to abstract classes, but you don't have to wast your single inheritance opportunity.

How many times have you stared at java code like this:

~~~ java
if (p1.compareTo(p2) <= 0)
~~~

Why not spell it out, to make the code more readable:

~~~ java
if (p1.lessThanOrEquals(p2))
~~~

We seldom do that, because implementing 4 methods, `lessThan`, `lessThanOrEquals`, `greaterThan` and `greaterThanOrEquals`, in each and every class that needs to be compared is too much work. We stick to the cryptical `compareTo`.

What if I say you only have to implement them once and then you can easily mixin the methods in each and every class that needs to be compared or sorted in some way.

I hope I don't have to elaborate on how bad idea it is to try to use inheritance (base class) for these kind of reusable pieces of code.

In Sculptor's textual DSL model traits are defined like this:

~~~
Trait Ordered {
    def boolean greaterThan(@Ordered other);
    def boolean greaterThanOrEquals(@Ordered other);
    def boolean lessThan(@Ordered other);
    def boolean lessThanOrEquals(@Ordered other);
    def abstract protected int compare(@Ordered other);
}
 
Entity Product with Ordered {
    String name
}
~~~

After code generation this means that you can implement the 4 comparison methods once, in the `Ordered` trait, and they will be available in `Product` and other domain objects that are defined `with Ordered`. The compare method must still be implemented in `Product`, because it is there you know what to compare with. The internal, generated, implementation is based on delegation, i.e. no magic.

Let us define another trait, which illustrates that traits also can hold state.

~~~
Trait PriceTag {
    - protected Money normalPrice
    protected double discount
    def Money price;
    def Money price(String currency);
}
 
Entity Product with PriceTag {
    String name
}
 
BasicType Money with Ordered {
    BigDecimal amount
    String currency
}
~~~

This means that `normalPrice` and `discount` will be mixed in to `Product`. You implement the `price` methods in the `PriceTag` trait. Product and all other domain objects that are defined `with PriceTag` will have those fields and methods.

Wouldn't it be nice to be able to compare product by price? Let us do that by combining the two traits. First add the compare method in `PriceTag`, so that you only have to implement it at one place.

~~~
Trait PriceTag {
    - protected Money normalPrice
    protected double discount
    def Money price;
    def Money price(String currency);
    def protected int compare(Ordered other);
}
~~~

Then mixin both traits into `Product`:

~~~
Entity Product with Ordered with PriceTag {
    String name
}
~~~

That's it. We have designed products with a rich price and compare interface.
Note that compareTo is no longer implemented in Product, only in PriceTag.

Try this new feature in [Sculptor 2.0.0][3].

   [1]: https://www.scala-lang.org
   [2]: https://www.scala-lang.org/node/126
   [3]: /2011/03/13/sculptor-20-is-out
