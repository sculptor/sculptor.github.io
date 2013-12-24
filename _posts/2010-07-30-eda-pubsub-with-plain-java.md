---
layout: post
title: "EDA pub/sub with plain java"
description: ""
category: 
tags: [Sculptor,EDA,Scalability]
author: Andreas Källberg
navbar_name: blog
---
{% include JB/setup %}

In a couple of posts we have seen how to [publish][1] and [subscribe][2] to events. I showed how to do that through the DSL in the model. But you have of course also the possibility to do pub/sub by plain java code as well. The key thing is to have a handle to the event bus.

The easiest way to explain it is to show an example, so, a simple service that publish an event as part of it's business logic:

~~~ java
@Service
public class GroundControlService {

    @Autowired
    @Qualifier("eventBus")
    private EventBus eventBus;

    @Override
    public void receive(Event event) {
        if (event instanceof BigLandingSuccess) {
            eventBus.publish("earthChannel", new Celebration("We did it!"));
            this.bringOutTheChampagne(ALL);
            eventBus.publish("supplierChannel", new MissingResource("Champagne"));
        }
    }
}
~~~

And for the other end, the subscriber:

~~~ java
@Service
public class SupplierService {

    @Autowired
    @Qualifier("eventBus")
    private EventBus eventBus;

    public void init() {
        eventBus.subscribe("supplierChannel", new EventSubscriber() {

            @Override
            public void receive(Event event) {
                if (event instanceof MissingResource) {
                    raceForContract(event);
                }
            }
        });
    }
}
~~~

Not so hard.

Now, here and there I've used the term 'event bus', and you have also seen it in java code above. Next time we will look closer into the bus. Why is there an abstraction and what can you do with it?

   [1]: /2010/07/26/eda-a-simple-example-with-sculptor
   [2]: /2010/07/29/eda-a-simple-example-with-sculptor-continued
