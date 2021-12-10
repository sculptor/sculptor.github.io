---
layout: post
title: "EDA CQRS Betting Sample"
description: ""
category: 
tags: [EDA,CQRS,Sculptor]
author: Patrik Nordwall
navbar_name: blog
---
{% include JB/setup %}

For todays [Xtext/Sculptor][1] seminar we have prepared a new example that illustrates some of the new event-driven features in Sculptor. It highlights a good case for using an architecture in line with [Command and Query Responsibility Segregation][2] (CQRS) pattern.

Consider an online betting system. It receives a massive amount of bets. A simple model for that could be:

~~~
ValueObject Bet {
    String betOfferId
    String customerId
    Double amount

    Repository BetRepository {
        save;
    }
}

Service BettingEngine {
    inject @BetRepository

    placeBet(@Bet bet);
}
~~~

The `BettingEngine` process the `Bet` and stores the information.

Questions pop up:

  * How many bets have been done by customer A?
  * Which customers place the higest bets, on average?
  * What are the top 10 high stakes?

It would be rather easy to develop support for those kind of queries in the master betting engine domain, but problems will soon bubble up:

  * *Poor performance* - the structure of the domain model is not optimized for all types of queries.
  * *Not scalable* - single centralized database will become a bottleneck.
  * *Hard to change* - too much functionality in one monolithic system.

Command-Query Responsibility Segregation (CQRS) comes to the rescue. Simplified it is about separating commands (that change the data) from the queries (that read the data). Separate subsystems take care of answering queries (reporting) and the domain for processing and storing updates can stay focused. The result of the commands are published to query subsystems, each optimized for its purpose.

Back to the betting sample. We are aiming for a design as illustrated in next drawing. Command side on the left (green) and Query side on the right (blue).

![Betting Demo Design][3] \\
<small>_Figure 1. Betting demo design_</small>

We publish domain events when bets are placed. There are several ways to do that in Sculptor, but one easy way is to mark a Service operation with the `publish` keyword in the model.

~~~
DomainEvent BetPlaced {
    - Bet bet
}

Service BettingPublisher {
    publishEvent(@BetPlaced betEvent) publish to jms:topic:bet;
}
~~~

Then, in the hand-written java code simply invoke this method. In this case a one-liner in `BettingEngine`.

That's all for the command side, now let us take a look at the query side. We define a new module or even better a completely new business component.

~~~
Module customer {
    Consumer BettingConsumer {
        inject @CustomerStatisticsRepository
        subscribe to jms:topic:bet
    }
    
    Service BettingQueryService {
        getHighBetters => CustomerStatisticsRepository.findHighAverageCustomers;
    }
    
    Entity CustomerStatistics {
        gap
        String customerId key
        int numberOfBets
        double averageAmount index
    
        Repository CustomerStatisticsRepository {
            findByKey;
            save;
            List findHighAverageCustomers(double limit);
            protected findByCondition;
        }
    }
}
~~~

We receive the events published from the betting engine by defining the subscribe keyword in the `BettingConsumer`.

In the query side we use domain objects that are optimized for the views and reports that are needed. We denormalize the data to minimize the number of joins needed when retrieving the data. Data is calculated ahead of time. In this case we calculate the average amount for the `CustomerStatistics`.

Since we are using publish/subscribe via a message bus, it might take a while until the query side is updated with the latest data, but that is not a problem. Most systems can be eventually consistent on the query side.

## Source Code

The full source code for the example is available here: [https://github.com/sculptor/sculptor/tree/master/sculptor-examples/eda-samples/sculptor-betting][4]

The slides of the presentation are available here: [https://www.slideshare.net/patriknw/sculptor][5]


## Summary

In the example we have chosen MongoDB as persistence store, Camel together with ActiveMQ as event bus. It is a matter of simple configuration to use something else, such as Oracle with JPA/Hibernate and Spring Integration for the event bus.

I was pretty impressed myself when I counted the number of lines of code that was required to implement this example. 70 lines! 40 in the model and 30 lines of hand written java code. User interface not counted.

   [1]: https://jwsdsl09sep.eventbrite.com/
   [2]: https://cqrs.wordpress.com/
   [3]: /images/2010-09-09-eda-cqrs-betting-sample/betting_demo.png
   [4]: https://github.com/sculptor/sculptor/tree/master/sculptor-examples/eda-samples/sculptor-betting
   [5]: https://www.slideshare.net/patriknw/sculptor
