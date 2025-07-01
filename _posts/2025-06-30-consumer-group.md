---
layout: post
title: "Implementing 3 consumers in a consumer group and viewing rebalancing"
categories: [general, kafka]
tags: [kafka, data, cloud, tools, streaming, python]
description: "How I built 3 consumers in a consumer group using Kafka-Python"
---

# Introduction

In this article, I want to show how to deploy multiple consumers in a consumer group and put in evidence the concept of rebalancing.

## Pre-requisites

### Python Dependencies

Following the same idea with the docker container to isolate our dev environment, we will create and activate a virtual environment:

```bash
python3 -m venv kafka-env
source kafka-env/bin/activate # On Windows: kafka-env\Scripts\activate
```

Then install the required package:

```bash
pip install kafka-python
```

## First steps

First we will have to create another kafka topic with 3 partitions this time.

Run this command below to do so

```
docker compose exec kafka kafka-topics --create \
    --topic web-events-partitioned \
    --bootstrap-server localhost:9092 \
    --partitions 3 \
    --replication-factor 1
```

We can then check if it has indeed 3 partitions

```
docker compose exec kafka kafka-topics --describe \
    --topic web-events-partitioned \
    --bootstrap-server localhost:9092
```

In the output, we should end up with something like this, as per the screenshot below:

<img src="/assets/media/30-06-consumer-group-lab/partitioned-topic.png">

## Python scripts

The python scripts are available in my github page, in order not to overload this article, I have decided not to integrate them in this page.
Feel free to check the code source <a href="https://github.com/thomaswong25520/kafka-code/tree/main/02-group_demo_consumers">here</a>.

## Testing phase

1 - First let's run our consumer by executing the following command `python3 consumer_group_partitioned.py`

2 - Then, let's run the producer script
`python3 partitioned_producer.py`

3 - Then, we can monitor in real time what is being pushed to our kafka broker

```
docker compose exec kafka kafka-console-consumer \
    --bootstrap-server localhost:9092 \
    --topic web-events-partitioned \
    --property print.key=true \
    --property print.partition=true \
    --property print.offset=true \
    --from-beginning
```

## Creating the partitioned topics

```
chmod +x setup_partitioned_topic.sh
./setup_partitioned_topic.sh
```

## Start the consumer group (in one terminal)

`python3 consumer_group_partitioned.py`

## Start the producer (in another terminal):

`python3 partitioned_producer.py`

#### Group consumer demo

In the demo video below, we can see the bottom right terminal shows different output in different color, each color representing a consumer.

This put in evidence the concept of consumer group, each consumer will be given a partition (resp 0, 1 or 2) to handle the traffic load

<div class="video-demo">
  <video autoplay loop muted playsinline>
    <source src="/assets/media/30-06-consumer-group-lab/group-consumer-1.webm" type="video/webm">
    <source src="/assets/media/30-06-consumer-group-lab/group-consumer-1.mp4" type="video/mp4">
    Your browser doesn't support video playback.
  </video>
</div>

## Rebalancing

Ok now let's see what happens in kafka broker and consumers side when a consumer is shut down or timeouts

The theory says the load will be rebalanced among the remaining consumers, let's put that in evidence

For this, we will need to re-create the consumers because the current one are being lunched via the same script and we want to shut down consumers one by one.

For this, I have made this <a href="https://github.com/thomaswong25520/kafka-code/blob/main/03-rebalancing/single_consumer.py">script</a>. to launch a single consumer and will launch 3 consumers, each consumer being launched via a single terminal

Then, after cloning the github repo, run this script

`python3 single_consumer.py`

Now, the idea is to monitor by running the following command on a terminal

```
bash

watch -n 2 'docker compose exec kafka kafka-consumer-groups --describe \
    --group web-events-group \
    --bootstrap-server localhost:9092'
```

This command monitors our kafka cluster by refreshing it every 2 seconds.

We can observe the rebalancing of traffic by keeping an eye on the `CONSUMER_ID` column, each line has a unique `CONSUMER_ID` and when a consumer is killed, the `CONSUMER_ID` will be given to another consumer

#### We can see the results in the demo below

<div class="video-demo">
  <video autoplay loop muted playsinline>
    <source src="/assets/media/30-06-consumer-group-lab/rebalancing-demo.webm" type="video/webm">
    <source src="/assets/media/30-06-consumer-group-lab/rebalancing-demo.mp4" type="video/mp4">
    Your browser doesn't support video playback.
  </video>
</div>

## Conclusion

We have put in evidence how consumer group work and how rebalancing work too in case a consumer shuts down / timeouts
