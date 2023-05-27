# Flow Programming Challenge

[![Epic IO](https://img.shields.io/badge/AIoT-Epic%20IO-blue)](https://epicio.com/)
[![Apache Kafka](https://img.shields.io/badge/streaming_platform-Apache%20Kafka-black.svg?style=flat-square)](https://kafka.apache.org)
[![Apache NiFi](https://img.shields.io/badge/ETL-Apache%20NiFi-yellowgreen)](https://nifi.apache.org/)

## Description

This challenge should be fully containerised and versioned, and it must be uploaded to a repository with its respective README.md with the deployment instructions. Must be plug and play to be evaluated.

You will need [Docker](https://docs.docker.com/install/) and [Docker Compose](https://docs.docker.com/compose/) to solve it.

The challenge has two parts, one oriented to processing data, and the other for visualizing of them.

## Part One: Backend

For processing data, create a NiFi Flow that consumes car detections from a Kafka topic called `epic.detections`, index them as a telemetry in database of your preference, generate alerts based on some attributes, and publish them into another Kafka topic called `epic.alerts`.

Examples detection message:

```json
{'Year': 2011, 'Make': 'Toyota', 'Model': 'Land Cruiser', 'Category': 'SUV'}
{'Year': 2020, 'Make': 'Mercedes-Benz', 'Model': 'C-Class', 'Category': 'Convertible, Sedan, Coupe'}
```

1. Each detection must be processed and added a new attribute called `Slug`, which consists of a slugfied string based on the following attributes: `Make`, `Model`, `Category`.

```json
{'Year': 2011, 'Make': 'Toyota', 'Model': 'Land Cruiser', 'Category': 'SUV', 'Slug': 'toyota-land-cruiser-suv'}
{'Year': 2020, 'Make': 'Mercedes-Benz', 'Model': 'C-Class', 'Category': 'Convertible, Sedan, Coupe', 'Slug': 'mercedes-benz-c-class-convertible-sedan-coupe'}
```

2. Index the messages processed as telemetry in a database of your choice. Justify your choice.

3. Generate alerts based on the following criteria:
    
    - All Toyota SUV vehicles older than 2010 must be published into the `epic.alerts` Kafka topic.

## Part Two: Frontend

For visualizing data, create a Dashboard in Node-RED with the following features
    - Pop-up notifications based on the generated alerts from `epic.alerts` Kafka topic.
    - A widget that displays a histogram based on the vehicle counters by year.

___

## Infra Setup

You simply need to create a Docker network called `epic-net` to enable communication between the services.

## Producers Setup

- Spin up the local single-node Kafka cluster (will run in the background):

```bash
$ docker-compose -f docker/docker-compose.kafka.yml up -d
```

- Check the cluster is up and running (wait for "started" to show up):

```bash
$ docker-compose -f docker/docker-compose.kafka.yml logs -f broker | grep "started"
```

- Start the detections producer (will run in the background):

```bash
$ docker-compose -f docker/docker-compose.producer.yml up -d
```

## How to watch the broker messages

Show a stream of detections in the topic `epic.detections` (optionally add `--from-beginning`):

```bash
$ docker-compose -f docker/docker-compose.kafka.yml exec broker kafka-console-consumer --bootstrap-server localhost:9092 --topic epic.detections
```

Topics:

- `epic.detections`: raw generated detections
- `epic.alerts`: generated alerts based on some criteria.

## Teardown

To stop the detections producer:

```bash
$ docker-compose -f docker/docker-compose.producer.yml down
```

To stop the Kafka cluster (use `down`  instead to also remove contents of the topics):

```bash
$ docker-compose -f docker-compose.kafka.yml stop
```

To remove the Docker network:

```bash
$ docker network rm epic-net
```
