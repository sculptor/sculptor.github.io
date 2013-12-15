---
layout: post
title: "Introducing a Type"
description: ""
category: 
tags: [Sculptor,DDD,OO,JPA,DSL,Productivity]
author: Patrik Nordwall
navbar_name: blog
---
{% include JB/setup %}

An important building block when creating a high quality domain model is to create small type objects. In this article we will create a Length type for the diameter of the Planet of the [helloworld application][1].

Length is a typical [Quantity][2] with a value and unit, e.g. meter, kilometer.

In the design model it looks like this:

~~~
BasicType Length {
    BigDecimal value min="0"
    -@LengthUnit unit
}

enum LengthUnit {
    cm, m, km
}

Entity Planet {
    gap
    scaffold
    String name key
    Long population min="0"
    -@Length diameter nullable
    -Set moons opposite planet

    Repository PlanetRepository {
        findByKey;
    }
}
~~~

We also need to convert between different units. The behaviour expressed as a JUnit test:

~~~ java
public class LengthTest {

    @Test
    public void shouldConvertFromMeterToKilometer() {
        Length length = new Length(new BigDecimal("31000"), m);
        Length lengthInKilometer = length.to(km);
        assertEquals(new Length(new BigDecimal("31"), km),
        lengthInKilometer);
    }

    @Test
    public void shouldConvertFromKilometerToMeter() {
        Length length = new Length(new BigDecimal("44"), km);
        Length lengthInMeter = length.to(m);
        assertEquals(new Length(new BigDecimal("44000"), m),
        lengthInMeter);
    }

    @Test
    public void shouldNotConvertSameUnit() {
        Length length = new Length(new BigDecimal("17"), km);
        Length length2 = length.to(km);
        assertSame(length, length2);
    }
}
~~~

[BasicType objects][3] may contain business logic in the same way as other [domain objects][7]. The following screencast illustrates how to implement the conversion.

BasicType is a stored in the same table as the Domain Object referencing it. It corresponds to JPA `@Embeddable`.

There are a lot of cases when it is a good idea to introduce types.

  * Identifers, natural business keys. It is more readable to pass around an identifier type instead of a plain `String` or `Integer`
  * [Money][4]
  * [Range][5]
  * [Quantity][2]

I can recommend reading [When to Make a Type][6], Martin Fowler.

   [1]: /documentation/hello-world-tutorial
   [2]: http://martinfowler.com/eaaDev/quantity.html
   [3]: /documentation/advanced-tutorial#basictype
   [4]: http://martinfowler.com/eaaCatalog/money.html
   [5]: http://martinfowler.com/eaaDev/Range.html
   [6]: http://www.martinfowler.com/ieeeSoftware/whenType.pdf
   [7]: /documentation/advanced-tutorial#domain-objects
