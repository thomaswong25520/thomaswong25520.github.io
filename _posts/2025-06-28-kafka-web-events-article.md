---
layout: post
title: "Test Building a Real-Time Web Events Pipeline with Kafka and Python"
categories: [general, kafka]
tags: [kafka, data, cloud, tools, streaming, python]
description: "A hands-on guide to building a web events Kafka data pipeline with Python - from setup to real-time processing"
---

# Building a Real-Time Web Events Pipeline with Kafka and Python

Ever wondered how major platforms like Netflix, LinkedIn, or Spotify handle millions of user interactions in real-time? The answer often involves **Apache Kafka**. Today, I'll walk you through building your own web events data pipeline using Kafka and Python.

By the end of this article, you'll have a complete understanding of how producers send events and consumers process them.

## Setting Up Our Kafka Infrastructure

First things first - let's create our Kafka server and topic. Think of a topic as a category where we'll store all our web events (clicks, page views, purchases, etc.).

### Create the Kafka server and the kafka topic

{% highlight bash %}
docker compose exec kafka kafka-topics --create \
 --topic web-events \
 --bootstrap-server localhost:9092 \
 --partitions 1 \
 --replication-factor 1
{% endhighlight %}

**Breaking down this command:**

- `docker compose exec kafka` - Execute command inside our Kafka container
- `kafka-topics --create` - Create a new topic
- `--topic web-events` - Name our topic (think of it as a category for events)
- `--bootstrap-server localhost:9092` - Connect to our Kafka server
- `--partitions 1` - Single partition for simplicity (production would use more)
- `--replication-factor 1` - No replication since we have only one broker

This creates a topic called `web-events` where we'll stream all our user interactions.

### Monitoring Our Events in Real-Time

Want to peek behind the curtain? This command lets you see events flowing into your topic as they happen:

{% highlight bash %}
docker compose exec kafka kafka-console-consumer \
 --bootstrap-server localhost:9092 \
 --topic web-events \
 --from-beginning
{% endhighlight %}

**Command breakdown:**

- `kafka-console-consumer` - Built-in tool to read messages
- `--topic web-events` - Read from our specific topic
- `--from-beginning` - Start from the first message ever sent (great for testing)

It's like having a live feed of everything happening on your platform. Leave this running in a terminal - you'll see events appear as they're produced!

## The Producer: Simulating Real Web Traffic

Let's simulate realistic web events - imagine users clicking buttons, viewing pages, making purchases, all happening simultaneously across your website.

### Python script to simulate real time web events

<img src="/assets/media/27-06-web-events-pipeline/kafka-producer-code">

Now, our Python producer generates realistic web events and streams them to Kafka in real-time. Each event contains structured data about user interactions - exactly what you'd see on a real website.

**Watch this in action:**

<div class="video-demo">
  <video autoplay loop muted playsinline>
    <source src="/assets/media/27-06-web-events-pipeline/kafka-producer-events-simulation.webm" type="video/webm">
    <source src="/assets/media/27-06-web-events-pipeline/kafka-producer-events-simulation.mp4" type="video/mp4">
    Your browser doesn't support video playback.
  </video>
</div>

**What you're seeing:** On the right window, our producer is generating web events (user clicks, page views, etc.) every few seconds. On the left window, you can see these events being stored as JSON data in our Kafka topic. Notice how each event is timestamped and contains realistic user interaction data.

This simulation represents what happens millions of times per day on platforms like Amazon or Facebook - user actions being captured and streamed for real-time processing.

## The Consumer: Processing Events as They Arrive

Now let's build the other side of the equation - a consumer that processes these events in real-time. This is where you'd typically run analytics, update dashboards, trigger notifications, or feed machine learning models.

### Python script to run the client listening to real-time web events

<img src="/assets/media/27-06-web-events-pipeline/kafka-consumer-code.png">

Then run the command to execute the script

{% highlight yaml %}
python3 simple_consumer.py
{% endhighlight %}

Our consumer continuously listens to the Kafka topic and processes each event as it arrives.

In a real application, this might update user profiles, calculate real-time metrics, or trigger personalized recommendations.

**See the consumer in action:**

<div class="video-demo">
  <video autoplay loop muted playsinline>
    <source src="/assets/media/27-06-web-events-pipeline/kafka-consumer-events-simulation.webm" type="video/webm">
    <source src="/assets/media/27-06-web-events-pipeline/kafka-consumer-events-simulation.mp4" type="video/mp4">
    Your browser doesn't support video playback.
  </video>
</div>

**What you're observing:** The consumer (lower right window) is reading events from our `web-events` topic (left terminal window) who had been fed data by our producer script (upper right window). Each event is parsed and could trigger various business logic - updating user sessions, calculating metrics, or feeding real-time dashboards.

## Why This Architecture Matters

This simple pipeline demonstrates the power of event-driven architecture:

- **Scalability**: Add more consumers to handle increased load
- **Reliability**: Kafka ensures no events are lost, even if consumers go down
- **Flexibility**: Multiple consumers can process the same events for different purposes
- **Real-time**: Events are processed as they happen, not in batches

## What's Next?

This basic pipeline is just the beginning. In production, you might:

- Add more sophisticated event schemas
- Implement multiple consumer groups for different use cases
- Add stream processing with Kafka Streams or Apache Flink
- Integrate with data warehouses or real-time dashboards
