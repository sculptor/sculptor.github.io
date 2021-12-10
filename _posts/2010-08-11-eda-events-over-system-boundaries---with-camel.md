---
layout: post
title: "EDA events over system boundaries   with camel"
description: ""
category: 
tags: [Sculptor,EDA,Scalability,Integration]
author: Andreas Källberg
navbar_name: blog
---
{% include JB/setup %}

[Last time][1] we examined how we could make our events travel over system boundaries with the help of [JMS][2] and [spring-integration][3].

Today we will do exactly the same but we will use [Apache Camel][4] as event engine instead. The task is to enable application domain events over system boundaries through JMS. If you don't remember our model, here it is again:

~~~
Application Universe {
    basePackage=org.helloworld

    Module milkyway {
        Service PlanetService {
            @BigLandingSuccess landOnPlanet(String planetName, String astronautName) publish to milkywayChannel;
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

To use Apache Camel as event engine, you need to specify it in the `sculptor-generator.properties` file:

~~~
integration.product=camel
~~~

And add the needed dependencies to the projects POM file:

~~~ xml
<dependency>
    <groupId>org.apache.camel</groupId>
    <artifactId>camel-core</artifactId>
    <version>2.12.2</version>
</dependency>
<dependency>
    <groupId>org.apache.camel</groupId>
    <artifactId>camel-jms</artifactId>
    <version>2.12.2</version>
</dependency>
<dependency>
    <groupId>org.apache.camel</groupId>
    <artifactId>camel-spring</artifactId>
    <version>2.12.2</version>
</dependency>
<dependency>
    <groupId>org.apache.activemq</groupId>
    <artifactId>activemq-camel</artifactId>
    <version>5.9.0</version>
</dependency>
<!-- xbean is required for ActiveMQ broker configuration in the spring xml file -->
<dependency>
    <groupId>org.apache.xbean</groupId>
    <artifactId>xbean-spring</artifactId>
    <version>3.16</version>
</dependency>
<dependency>
    <groupId>javax.xml.bind</groupId>
    <artifactId>jaxb-api</artifactId>
    <version>2.2.11</version>
</dependency>
~~~

Regenerate and you will have a new `camel.xml` file in `src/main/resources/`. This file is only generated once, and you can use it to add configuration for Camel.

To do the same as we did with spring-integration, i.e. put a domain event on a JMS topic, we just have to add route rule to the `camel.xml` file:

~~~ xml
<camel:route>
    <camel:from uri="direct:shippingChannel"/>
    <camel:to uri="jms:topic:shippingEvent"/>
</camel:route>
~~~

And that's it. Now we are publishing our domain event to an ActiveMQ topic. Yes, a bit strange that it works, but it gets clearer if we look in `camel.xml` and see whats was already there:

~~~ xml
<bean id="camelEventBusImpl" class="org.sculptor.framework.event.CamelEventBusImpl"/>
<alias name="camelEventBusImpl" alias="eventBus"/>
 
<camel:camelContext id="camel">
    <camel:package>org.sculptor.shipping</camel:package>
    <camel:template id="producerTemplate"/>
</camel:camelContext>

<!-- Camel ActiveMQ to use the ActiveMQ broker -->
<bean id="jms" class="org.apache.activemq.camel.component.ActiveMQComponent">
    <property name="brokerURL" value="tcp://localhost:61616"/>
</bean>
~~~

We have a bean that wires the JMS api to the actual ActiveMQ instance together. So you have to have an ActiveMQ instance up and running listening on port 61616.

All for today, next time...we'll see what the subject is...


   [1]: /2010/08/02/eda-events-over-system-boundaries
   [2]: https://en.wikipedia.org/wiki/Java_Message_Service
   [3]: https://springsource.org/spring-integration
   [4]: https://camel.apache.org/
