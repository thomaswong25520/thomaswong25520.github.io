---
layout: post
title: Building Your First Kafka Data Pipeline - A Step-by-Step Guide
categories: [general, kafka, tutorial]
tags: [kafka, data, tutorial]
description: I decided to build my own Kafka pipeline and wanted to document the process. If you’re interested, here’s how I did it and what I learned along the way.
---

Step 1: Get Kafka
Download the latest Kafka release and extract it:

{% highlight yaml %}
$ tar -xzf kafka_2.13-4.0.0.tgz
$ cd kafka_2.13-4.0.0
{% endhighlight %}

Step 2: Start the Kafka environment

Using JVM Based Apache Kafka Docker Image

Get the Docker image:
{% highlight yaml %}
docker pull apache/kafka:4.0.0
{% endhighlight %}

Start the Kafka Docker container:
{% highlight yaml %}
docker run -p 9092:9092 apache/kafka:4.0.0
{% endhighlight %}

Step 3: Create a topic to store your events

Note: since we are working in a Docker container, all commands starting with /bin needs to be replaced by /opt/kafka/

{% highlight yaml %}
bin/kafka-topics.sh --create --topic quickstart-events --bootstrap-server localhost:9092
{% endhighlight %}

{% highlight yaml %}
bin/kafka-topics.sh --describe --topic quickstart-events --bootstrap-server localhost:9092
{% endhighlight %}
