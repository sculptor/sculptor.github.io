---
layout: post
title: "EDA events over system boundaries"
description: ""
category: 
tags: [Sculptor,EDA,Scalability,Integration]
author: Andreas Källberg
navbar_name: blog
---
{% include JB/setup %}

In the last [post][1] we explained why we have the 'event bus' notion. And earlier we have seen how to use it to publish and subscribe, both through the DSL in the model, or through plain java code.

Today we thought we should look how you can make domain events part of the public api of your business component and publish them to the rest of the world.

To do this we will use the event bus implementation that is based on spring integration. We will use its [JMS outbound channel adapter][2] to publish the `BigLandingSuccess` domain event to a jms topic that the rest of the world can listen to.

If you don't remember, here is the model:

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

But first, how do we replace the default event bus implementation with the [spring-integration][5] based?

In `sculptor-generator.properties`, add the property:

~~~
integration.product=spring-integration
~~~

And in the project POM file add the dependency:

~~~ xml
<dependency>
   <groupId>org.springframework.integration</groupId>
   <artifactId>spring-integration-jms</artifactId>
   <version>2.2.6.RELEASE</version>
</dependency>
~~~

Regenerate, and you are home. So, now all event routing will use spring-integration as engine. So without any other changes, it works exactly as before.
Now you have to decide how your public api should look like, and in our case we decides that the `BigLandingSuccess` domain event should be made available for other applications to listen to.

And now we take advantage of the power of spring-integration. When we regenerated with spring-integration enabled, we ended up with the file `src/main/resources/spring-integration.xml`.

This is the place where all spring-integration definitions lands. It is generated once, so you are now free to edit it. Open the file and add:

~~~ xml
<jms:outbound-channel-adapter id="publicMilkywayChannel"  destination="publicMilkywayTopic" channel="milkywayChannel"/>
~~~

You will need a [Spring JMS bean][3] named 'publicMilkywayTopic' and that bean needs to map to an actual message broker instance, for example [ActiveMQ][4]. But the spring bean and message broker setup is out scope for this blog entry, so that one we leave to you.

The only caveat now is that we are exposing our java objects in the message. This can be cured with a transformation step before the JMS adapter. So add a transformer to your spring-integration configuration:

~~~ xml
<object-to-string-transformer input-channel="milkywayChannel" output-channel="milkywayMessagesAsStringChannel"/>
~~~

And change the JMS outbound channel to wire up with the new channel:

~~~ xml
<jms:outbound-channel-adapter id="publicMilkywayChannel"  destination="publicMilkywayTopic" channel="milkywayMessagesAsStringChannel"/>
~~~

Spring-integration has a lot of transformers you can use. The above just transforms a java object to a string through its `toString` method. That is of course not always what you want, but it works for this example.

So that was all that had to be done to make a domain event part of your public API. Of course, you need to have proper documentation to give others a fair chance of finding your event.

If we take one step back and consider what we have done.
First, we declared the `BigLandingSuccess` domain event and publishes it internally.

Second, we add an adapter that listens to the channel and in its turn publishes it on a JMS topic, i.e. makes it public.

Third, we added a transformation step before doing a public publish to remove the dependency to our classes.

Now, this is nice, isn't it? We can have a lot of domain events internally in our application and by that take advantage of all the nice attributes of EDA. And we can by choice make a domain event public with 'just' configuration changes.

Quite long post, but we got pretty much done. Next time we will look how to do the same with [Apache Camel][6].


   [1]: /2010/08/01/eda-why-the-event-bus-in-sculptor
   [2]: http://docs.spring.io/spring-integration/reference/html/jms.html#jms-outbound-channel-adapter
   [3]: http://static.springsource.org/spring/docs/3.0.x/reference/jms.html
   [4]: http://activemq.apache.org/
   [5]: http://www.springsource.org/spring-integration
   [6]: http://camel.apache.org/
