---
layout: post
title:  "That Mysterious Latency"
date:   2017-07-19
herounit: PostAlleyGraffiti
---
"<i>The real cycle you're working on is a cycle called yourself. The machine that appears to be 'out there' and the person that appears to be 'in here' are not two separate things. They grow toward Quality or fall away from Quality together.<i>" - Robert M Pirsig: Zen And The Art of Motorcycle Maintenance

Working on distributed systems is a challenging job, and often we find ourselves in a situation where we grapple with the issues for which we have no clue at all. Its challenging not because of the system, but its because we overlook things which affect the quality of a component in the system, which eventually affects the entire system.

Few weeks back I was working on designing and implementing a new agent in our existing set of agents to perform pretty simple task: read a message off of the message queue, perform some action and delete the message. I implemented the agent, tested it in devo, got it reviewed and finally shipped it. As soon as the agent was deployed in production, we started seeing latency in our platform. At the same time there was another issue in our platform, due to which we had to rollback weeks worth of changes. We rolled-back the changes and created a top priority task in our sprint to investigate the root cause for the latency. Pretty usual course of action.

This mysterious latency made me question everything my team and I did during the past week. To add to the problem, there were some 200 changes in the packages which were not owned by my team. Now this was a daunting task, its like finding a needle in the haystack. Somebody had to do this, so why not me. How you eat an elephant? One piece at a time. 

I started by reverting all suspected packages (owned by us and other teams), and started adding them one by one. I added all the packages which were not ours, and checked the latency metric. Metrics looked perfectly fine. This made it clear that we introduced the latency not somebody else. Next, I started adding packages that we own. Now the problem here was, that in our package we had multiple commits. So I started adding individual package, commit by commit. I followed this since it will help me find that offending commit. I found the commit which caused the increase in latency. The new agent which I implemented caused the latency. But why? It's fairly simple agent with very simple functionality: read the message from the queue, take some action and delete the message.

Now that I have figured out that the SQS agent was the root cause, I started digging into the SQS's client code. And, I found that SQS AmazonSQSClient uses default Apache HttpClient with default configurations if you don't provide one.

{% highlight java %}
public AmazonSQSClient(AWSCredentials awsCredentials) {
    this(awsCredentials, configFactory.getConfig());
}
{% endhighlight %}

{% highlight java %}
protected ClientConfiguration getDefaultConfig() {
    return new ClientConfiguration();
}
{% endhighlight %}

I looked at the ClientConfiguration class, and therein one line of code caught my attention:

{% highlight java %}
/** The default max connection pool size. */
public static final int DEFAULT_MAX_CONNECTIONS = 50;
{% endhighlight %}

<b>'50'</b>, this magic number. I have seen this before, and I do remember where. Our agents are multi-threaded application and each agent is configured to spawn 50 threads to poll messages from the queue and process the message.

<i>What a blunder!</i> I was so stupid to overlook this and learned my lesson by humiliating myself by not considering the inherent distributed nature of our agents. After I added a new agent in our service, we had two agents (100 threads trying to access SQS) competing for the same pool of 50 connections, and hence the service experienced latency while interacting with SQS. One can ask me how come we did not catch this in devo. This was not an issue in devo, since, in devo we had our agents configured to spawn <b>10</b> threads. So adding new agent (20 threads accessing SQS) did not cause any latency.

Distributed systems are just like motorcycles. An incorrect configuration could affect the smoothness of any distributed system.
