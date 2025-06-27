---
layout: post
title: Kafka data pipeline - Web events
categories: [general, kafka]
tags: [kafka, data, cloud, tools]
description: I tried to build a web events kafka data pipeline with Python
---

### Create the Kafka server and the kafka topic

{% highlight yaml %}
docker compose exec kafka kafka-topics --create \
 --topic web-events \
 --bootstrap-server localhost:9092 \
 --partitions 1 \
 --replication-factor 1
{% endhighlight %}

### Check our kafka server to see the events stored in the topic

{% highlight yaml %}
docker compose exec kafka kafka-console-consumer \
 --bootstrap-server localhost:9092 \
 --topic web-events \
 --from-beginning
{% endhighlight %}

### Python script to simulate real time web events

<img src="/assets/media/27-06-web-events-pipeline/kafka_producer.png">

<iframe width="420" height="315" src="/assets/media/27-06-web-events-pipeline/kafka-producer-events-simulation.mkv" frameborder="0" allowfullscreen></iframe>
