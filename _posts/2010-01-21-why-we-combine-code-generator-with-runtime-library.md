---
layout: post
title: "Why we Combine Code Generator with Runtime Library"
description: ""
category: 
tags: [Sculptor,Spring,Roo]
author: Patrik Nordwall
navbar_name: blog
---
{% include JB/setup %}

Sculptor is a development tool, but it also has a small runtime library. Some people find it scaring to bring in yet another library in their software stack.

It contains classes of utility and general character. It is not advanced at all. We could have generated those also and have no runtime dependency at all. Instead, we try to only generate things that must be variable, such as application specific structure and glue code. Static content should not be generated.

If we would have decided to not use a runtime library I think it would be a high risk that we would have generated code with bad design, such as repeated, duplicated code. We think in terms of DRY for the generated code. We always try to produce code as if we would have written it by hand, without a code generator. There are some exceptions to this rule, for example separation of generated and hand written code in separate classes.

It is interesting to compare Spring Roo and Sculptor. I see a lot of similarities, but also differences. One difference related to this topic is that Roo doesn't have a runtime library (except ordinary Spring Framework). They generate everything. They are using this as a selling point, but I think they have fallen into the trap of generating duplicated code. Take a look at the persistence methods in the ITD for an Entity.

~~~ java
@Transactional
public void Planet.remove() {
    if (this.entityManager == null)
        this.entityManager = entityManager();
    if (this.entityManager.contains(this)) {
        this.entityManager.remove(this);
    } else {
        Planet attached = this.entityManager.find(Planet.class, this.id);
        this.entityManager.remove(attached);
    }
}
~~~

What happens if you decide to remove Roo? Then you will have to maintain a lot of duplicated code, that was generated initially. I think it illustrates how easy it is to forget or not bother about ordinary design principles when you solve everything with code generation. DRY should of course be used even when you have a code generator in your hands.

In Sculptor we have placed the general persistence operations in [parameterized access objects][1], in the runtime library.

   [1]: /documentation/advanced-tutorial#generic-access-objects
