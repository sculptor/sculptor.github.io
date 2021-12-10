---
layout: post
title: "EDA a simple example with Sculptor"
description: ""
category: 
tags: [Sculptor,EDA,Scalability,DSL]
author: Andreas Källberg
navbar_name: blog
---
{% include JB/setup %}

Now we have gone through EDA in general, and briefly covered how we support it in Sculptor. It's time to see how it can be used.

We start with a simple example where we will show how to declare a domain event and how that event is published.

Let us use our old, well known, [hello world example][1].

~~~
Application Universe {
    basePackage=org.helloworld

    Module milkyway {
        Service PlanetService {
            void landOnPlanet(String planetName, String astronautName);
        }
    }
}
~~~

That is our model. So, what might the rest of the 'universe' be interested of here? Well, if NASA sent out a space ship with an astronaut on it who's mission was to land on some far far away planet, wouldn't they be interested in the event of a landing? I think so. So, when a astronaut lands on a planet, we would like the service to publish an event of this happening.

Lets start with re-defining the model with the event:

~~~
Application Universe {
    basePackage=org.helloworld

    Module milkyway {
        Service PlanetService {
            void landOnPlanet(String planetName, String astronautName);
        }

        DomainEvent BigLandingSuccess {
            String planetName
            String astronautName
        }
    }
}
~~~

DomainEvents may contain attributes and references in the same way as ValueObjects and Entities. DomainEvents are always immutable and not persistent.

Events are about something happening at a point in time, so it's natural for events to contain time information. Sculptor automatically adds two timestamps, occurred and recorded. The time the event occurred in the world and the time the event was noticed.

Ok, so now we have the event defined, now we want it to be published. The easiest way of doing this is to mark the service in the DSL (you can of course publish events programmatically, but more about that in a later blog entry) . So, once again, lets re-define our model:

~~~
Application Universe {
    basePackage=org.helloworld

    Module milkyway {
        Service PlanetService {
            @BigLandingSuccess landOnPlanet(String planetName, String astronautName)
                        publish to milkywayChannel;
        }

        DomainEvent BigLandingSuccess {
            String planetName
            String astronautName
        }
    }
}
~~~

The operation must return a DomainEvent or take a DomainEvent as parameter. That event is published to the defined channel when the operation has been invoked.

As an alternative the DomainEvent instance can be created from the return value or parameters. The DomainEvent must have a matching constructor.


The result of the above declarative way of publishing events is a generated annotation `@Publish` on the method. It will trigger the [Spring AOP][2] advice `PublishAdvice` that is part of Sculptor framework.


Recap: We have declared an event in our model, and further marked our service to publish this event when it happens. As an API, I think this is very neat. Everyone who is interested can see that when an astronaut lands on a planet, he or she can be notified.

And that is what we will talk about next time:

How can I be notified when things happens?


   [1]: /documentation/hello-world-tutorial
   [2]: https://docs.spring.io/spring/docs/3.0.x/reference/aop.html
