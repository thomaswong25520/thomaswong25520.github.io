---
layout: post
title: "Implementing Consumer Groups and Observing Kafka Rebalancing"
categories: [general, kafka]
tags: [kafka, data, cloud, tools, streaming, python]
description: "A practical guide to implementing consumer groups and understanding Kafka's rebalancing mechanism"
---

# Introduction

In this article, I'll demonstrate how to deploy multiple consumers within a consumer group and observe Kafka's automatic rebalancing mechanism in action. This builds upon my previous post about basic Kafka producers and consumers.

## Prerequisites

Before diving in, ensure you have Docker and Python installed on your system.

### Setting Up the Python Environment

Create and activate a virtual environment to keep dependencies isolated:

```bash
python3 -m venv kafka-env
source kafka-env/bin/activate  # On Windows: kafka-env\Scripts\activate

# Install the required package
pip install kafka-python
```

## Creating a Multi-Partition Topic

Unlike single-partition topics, we need multiple partitions to demonstrate consumer group behavior effectively.

```bash
# Create a topic with 3 partitions
docker compose exec kafka kafka-topics --create \
    --topic web-events-partitioned \
    --bootstrap-server localhost:9092 \
    --partitions 3 \
    --replication-factor 1

# Verify the partitions
docker compose exec kafka kafka-topics --describe \
    --topic web-events-partitioned \
    --bootstrap-server localhost:9092
```

You should see output confirming 3 partitions:

<img src="/assets/media/30-06-consumer-group-lab/partitioned-topic.png" alt="Kafka topic with 3 partitions">

## Implementation

The complete Python scripts are available on my [GitHub repository](https://github.com/thomaswong25520/kafka-code/tree/main/07-kafka-internals-cluster-membership).

- **`consumer_group_partitioned.py`**: Launches 3 consumers in a single process
- **`partitioned_producer.py`**: Produces messages with user IDs as keys for consistent partitioning
- **`setup_partitioned_topic.sh`**: Helper script to create the partitioned topic

## Running the Consumer Group Demo

### Step 1: Start the Consumer Group

```bash
python3 consumer_group_partitioned.py
```

### Step 2: Start the Producer

In a new terminal:

```bash
python3 partitioned_producer.py
```

### Step 3: Monitor the Message Flow (Optional)

To see messages being distributed across partitions:

```bash
docker compose exec kafka kafka-console-consumer \
    --bootstrap-server localhost:9092 \
    --topic web-events-partitioned \
    --property print.key=true \
    --property print.partition=true \
    --property print.offset=true \
    --from-beginning
```

### Demo: Consumer Group in Action

In the demo below, notice how each consumer (represented by a different color) handles exactly one partition. This demonstrates Kafka's load distribution:

<div class="video-demo">
  <video autoplay loop muted playsinline>
    <source src="/assets/media/30-06-consumer-group-lab/group-consumer-1.webm" type="video/webm">
    <source src="/assets/media/30-06-consumer-group-lab/group-consumer-1.mp4" type="video/mp4">
    Your browser doesn't support video playback.
  </video>
</div>

## Testing Kafka Rebalancing

Now for the interesting part: what happens when a consumer fails?

### Setting Up Individual Consumers

To properly test rebalancing, we need to run each consumer in its own process. I've created a [`single_consumer.py`](https://github.com/thomaswong25520/kafka-code/blob/main/03-rebalancing/single_consumer.py) script for this purpose.

Launch three consumers in separate terminals:

```bash
# Terminal 1
python3 single_consumer.py 1

# Terminal 2
python3 single_consumer.py 2

# Terminal 3
python3 single_consumer.py 3
```

### Monitoring the Rebalancing Process

In a fourth terminal, monitor the consumer group status:

```bash
watch -n 2 'docker compose exec kafka kafka-consumer-groups --describe \
    --group web-events-group \
    --bootstrap-server localhost:9092'
```

### Triggering a Rebalance

1. Kill one of the consumers (Ctrl+C)
2. Watch the monitoring terminal
3. After couple of seconds, Kafka detects the failure and reassigns the partition

Pay attention to the `CONSUMER-ID` column - each partition is initially assigned to a specific consumer.
When Consumer 2 (<span style="color:green">green</span>, with ID ending in `a9b`) is terminated, watch how Kafka automatically reassigns its partition. In this demo, partition 1 gets redistributed to Consumer 3 (<span style="color:blue">blue</span>, with ID ending in `86e`).
Subsequently, when Consumer 3 is terminated, both its original partition and the inherited one are reassigned to the remaining Consumer 1 (<span style="color:red">red</span>, with ID ending in `f6f`), demonstrating how a single consumer can handle multiple partitions when needed.

### Demo: Rebalancing in Action

<div class="video-demo">
  <video autoplay loop muted playsinline>
    <source src="/assets/media/30-06-consumer-group-lab/rebalancing-demo.webm" type="video/webm">
    <source src="/assets/media/30-06-consumer-group-lab/rebalancing-demo.mp4" type="video/mp4">
    Your browser doesn't support video playback.
  </video>
</div>

## Key Takeaways

1. **Automatic Load Distribution**: Kafka automatically assigns partitions to consumers in a group
2. **Fault Tolerance**: When a consumer fails, its partitions are redistributed to healthy consumers
3. **No Message Loss**: During rebalancing, no messages are lost - they're simply processed by different consumers
4. **Scalability**: You can add or remove consumers dynamically, and Kafka handles the rebalancing

## Conclusion

This demonstration shows the power of Kafka's consumer groups and automatic rebalancing.
In production environments, this mechanism ensures high availability and efficient resource utilization without manual intervention.
Whether you're handling millions of events or building a simple event-driven system, understanding these concepts is crucial for building robust streaming applications.
