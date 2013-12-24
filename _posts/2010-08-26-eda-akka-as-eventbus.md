---
layout: post
title: "EDA Akka as EventBus"
description: ""
category: 
tags: [Sculptor,EDA,Scalability,Akka]
author: Andreas Källberg
navbar_name: blog
---
{% include JB/setup %}

Some time since [last entry][1] in the EDA-sequence, but here we are again. Today we are going to do something really interesting. A friend and colleague of us, [Jonas Bonér][2], is the creator of a super interesting framework called [Akka][3].
Akka is an event-driven platform for constructing highly scalable and fault tolerant applications. It is built with [Scala][4], but also have a rich API for java. It follows the [Actor Model][5] and together with [Software Transactional Memory][6] (STM), it raises the abstraction level and provides an easy to use tool for building highly concurrent applications.

So, today we are going to take advantage of the java API in Akka to do our own `EventBus` implementation.

First, update your POM file with the dependencies needed for Akka:

~~~ xml
<dependency>
    <groupId>com.typesafe.akka</groupId>
    <artifactId>akka-actor_2.10</artifactId>
    <version>2.2.3</version>
</dependency>
~~~

Second, create your implementation of the bus, `AkkaEventBus.java`:

~~~ java
package org.foo;

import org.sculptor.framework.event.Event;
import org.sculptor.framework.event.EventBus;
import org.sculptor.framework.event.EventSubscriber;

import akka.actor.ActorRef;
import akka.actor.ActorRegistry;
import akka.actor.UntypedActor;
import akka.actor.UntypedActorFactory;

public class AkkaEventBus implements EventBus {

    public boolean subscribe(final String topic, final EventSubscriber subscriber) {
        UntypedActor.actorOf(new UntypedActorFactory() {
            public UntypedActor create() {
                return new ActorListener(topic, subscriber);
            }
        }).start();

        return true;
    }

    public boolean unsubscribe(String topic, EventSubscriber subscriber) {
        // TODO : implement mapping between arbitrary subscriber and actor in registry
        return true;
    }

    public boolean publish(String topic, Event event) {
        ActorRef[] actorsForTopic = ActorRegistry.actorsFor(topic);
        for (int i = 0; i < actorsForTopic.length; i++) {
            actorsForTopic[i].sendOneWay(event);
        }
        return true;
    }

    @SuppressWarnings("unchecked")
    private class ActorListener extends UntypedActor {
        final String topic;
        final EventSubscriber subscriber;
    
        ActorListener(String topic, EventSubscriber subscriber) {
            this.topic = topic;
            this.subscriber = subscriber;
            this.getContext().setId(topic);
        }
    
        @Override
        public void onReceive(Object message) throws Exception {
            this.subscriber.receive((Event) message);
        }
    }
}
~~~

As you can see, I lack the unsubscribe implementation and I left out equals and hashCode overrides, but that I leave to you.

Third, add it as our bus implementation through spring config:

~~~ xml
<bean id="akkaEventBus" class="org.foo.AkkaEventBus"/>
<alias name="akkaEventBus" alias="eventBus"/>
~~~

What we have done now is built an highly scalable and concurrent EventBus that dispatches event asynchronously.

Pretty easy, right? :-)

It doesn't run over the network, but Akka has some nice modules for that as well, so that is our task for next time.


   [1]: /2010/08/11/eda-events-over-system-boundaries---with-camel
   [2]: http://jonasboner.com/
   [3]: http://akka.io/
   [4]: http://www.scala-lang.org/
   [5]: http://en.wikipedia.org/wiki/Actor_model
   [6]: http://en.wikipedia.org/wiki/Software_transactional_memory
