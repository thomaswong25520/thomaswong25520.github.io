---
layout: post
title: "[KAFKA INTERNALS] - Understanding Kafka Cluster Membership: How Brokers Join and Leave the Cluster"
categories: [general, kafka]
tags: [kafka, data, cloud, tools, streaming, internals, cluster]
description: "Deep dive into Kafka's cluster membership mechanism, exploring how brokers register, maintain heartbeats, and handle failures in both ZooKeeper and KRaft modes"
---

## Understanding Kafka Cluster Membership: How Brokers Join and Leave the Cluster

In this article, I want to show how does Zookeeper keeps track of broker registering, leaving by leveraging the logs.

We will see how the first broker is set as being the Controller and also see how the partition leadership is being operated.

### First step

The first command we have to run is `docker compose up -d` which will launch our zookeeper service and our 3 kafka brokers.

## Controller election

When launching our services, intituively we could think broker 1 will be the controller.

In fact, this is not the case.

Looking at the respective kafka broker logs, we see that it is all about the broker who first successfully creates the zookeeper node (znode): you are right, this is a race!

broker 3 log:

`[2025-07-17 18:14:11,261] INFO [Controller id=3] 3 successfully elected as the controller. Epoch incremented to 1 and epoch zk version is now 1 (kafka.controller.KafkaController)`

broker 1 log:

`[2025-07-17 18:14:11,277] DEBUG [Controller id=1] Broker 3 was elected as controller instead of broker 1 (kafka.controller.KafkaController)`

broker 2 log:
`[2025-07-17 18:14:11,300] DEBUG [Controller id=2] Broker 3 was elected as controller instead of broker 2 (kafka.controller.KafkaController)`

We can see it only was a matter of ms:

- broker 3 created the znode at 18:14:11,261
- broker 1 tried to create it at 18:14:11,277
- broker 2 tried to create it at 18:14:11,300

This is also verified on the `/controller` namespace on zookeeper

<img src="/assets/media/17-07-cluster-membership/monitor-py.png">
