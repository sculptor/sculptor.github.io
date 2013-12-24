---
layout: post
title: "EDA a simple example with Sculptor continued"
description: ""
category: 
tags: [Sculptor,EDA,Scalability]
author: Andreas Källberg
navbar_name: blog
---
{% include JB/setup %}

Lets continue with our simple example from the last [post][1]. We declared a domain event in the model and we marked a service to publish that event to a channel.

Now, what about getting notified when such an event is published?

Continuing with our model from last time, its just a matter of editing it. Lets create another module with a service that is interested of the event:

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

    Module houston {
        Service GroundControlService {
            subscribe to milkywayChannel
            bringOutTheChampagne(int noOfBottles);
        }
    }
}
~~~

The result is that the `GroundControl` service will implement the `EventSubscriber` interface and be marked with `@Subscribe` annotation. That means that the `GroundControl` service will automatically be added as subscriber to `milkywayChannel`. It will be notified, receive method called, when events are published to that channel.


You will get a stub of the receive method of the `EventSubscriber` interface.

~~~ java
@Override
public void receive(Event event) {
    // TODO Auto-generated method stub
    throw new UnsupportedOperationException("receive not implemented");
}
~~~

So that you will need to implement with your logic:

~~~
@Override
public void receive(Event event) {
    if (event instanceof BigLandingSuccess) {
        this.bringOutTheChampagne(null, 999999);
    }
}
~~~

Ok, so now we covered some very simple notations for publishing and subscribing to events. Next time I'll show how to do the same but in plain java code.


   [1]: /2010/07/26/eda-a-simple-example-with-sculptor
