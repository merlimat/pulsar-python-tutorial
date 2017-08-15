---
title: Pulsar Python Tutorial.
description: Learn how to create Pulsar applications in Python.
tags:
  - pulsar
  - tutorial
  - python
  - producer
  - consumer
---

# Getting started with Pulsar in Python

In this tutorial, we will create two Python scripts that can send and receive
messages through a [Pulsar](https://pulsar.incubator.apache.org/) topic.

We will start a local Pulsar cluster and test the scripts in a live cluster.

## Start Pulsar standalone

A very convenient way to start playing with Pulsar is to start a standalone service. That includes
a complete Pulsar-cluster in a single JVM instance.

There are few ways to get a Pulsar standalone cluster up and running:
 * Using binary releases: [Local cluster documentation](https://pulsar.incubator.apache.org/docs/latest/getting-started/LocalCluster/)
 * Using Docker: [Pulsar in Docker](https://pulsar.incubator.apache.org/docs/latest/getting-started/docker/)

If you are in MacOS, you can even use [Homebrew](https://brew.sh/) to install a local Pulsar cluster
for testing purposes:

```shell
# Register tap
$ brew tap streamlio/homebrew-formulae

# Install
brew install streamlio/homebrew-formulae/pulsar

# Start the Pulsar standalone as a background service
brew services start pulsar
```

## Requirements for the Python client

The Pulsar Python client library can be installed directly from [PyPI](https://pypi.org/project/pulsar-client).

| OS    | Versions                               | Python versions         |
|:------|:---------------------------------------|:------------------------|
| Linux | Ubuntu, CentOS, Debian and most others | 2.7, 3.3, 3.4, 3.5, 3.6 |
| MacOS | 10.11 (El capitan), 10.12 (Sierra)     | 2.7, 3.6                |

```shell
$ pip install pulsar-client
```

## Clone the project

From the command line, use the following Git command to clone this project:

```bash
$ git clone https://github.com/streamlio/pulsar-python-tutorial.git
```

This command creates a directory named `pulsar-python-tutorial` at the current location, which contains
the tutorial basic Pulsar client Python project. The `pulsar-python-tutorial` directory
contains the following items:

File or directory | Description
:-----------------|:-----------
`consumer_tutorial.py` | Consumer tuorial code
`producer_tutorial.py` | Producer tuorial code


## Consumer

The first step is to start a consumer. This will create a subscription and make sure that every
message published after that is going to be retained by Pulsar, until an explicit acknowledgment
of the message processing is received.

The consumer code is available at [consumer_tutorial.py](consumer_tutorial.py).
The sample code is very simple:

```python
import pulsar

# Create a Pulsar client instance. The instance can be shared across multiple
# producers and consumers
client = pulsar.Client('pulsar://localhost:6650')

# Subscribe to the topic. If the topic does not exist, it will be
# automatically created
consumer = client.subscribe('persistent://sample/standalone/ns1/my-topic',
                            'my-subscription')

while True:
    try:
        # try and receive messages with a timeout of 10 seconds
        msg = consumer.receive(timeout_millis=10000)

        # Do something with the message
        print("Received message '%s'", msg.data())

        # Acknowledge processing of message so that it can be deleted
        consumer.acknowledge(msg)
    except Exception:
        print("No message received in the last 10 seconds")

client.close()
```

You can start the consumer with:

```shell
python consumer_tutorial.py
```


## Producer

The next step is to start the producer. In this example, we will start ingesting sample data into
a topic where the consumer has already created a subscription.The producer code is available at [producer_tutorial.py](producer_tutorial.py):

```python
import pulsar

# Create a Pulsar client instance. The instance can be shared across multiple
# producers and consumers
client = pulsar.Client('pulsar://localhost:6650')

# Create a producer on the topic. If the topic doesn't exist
# it will be automatically created
producer = client.create_producer(
                'persistent://sample/standalone/ns1/my-topic')

for i in range(10):
    content = 'hello-pulsar-%d' % i
    # Publish a message and wait until it is persisted
    producer.send(content)
    print('Published message: "%s"' % content)

client.close()
```

You can start the producer with:

```shell
python producer_tutorial.py
```

At this point you should see the consumer to start receiving the published sample messages.

You can find more documentation for Pulsar Python client at https://pulsar.incubator.apache.org/docs/latest/clients/Python/
 and the complete API reference at https://pulsar.incubator.apache.org/api/python/.
