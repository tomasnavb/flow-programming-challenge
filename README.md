
# Flow Programming Challenge - Tomás Navarro




## Description

This is my personal project made to participate in the "Flow Programming Challenge" from Epic IO.
In this document I'm going to explain how to deploy the project.


## Deployment

## Infra Setup

You will need [Docker](https://docs.docker.com/install/) and [Docker Compose](https://docs.docker.com/compose/) installed in your computer.

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

## Nifi Setup

- Start the Nifi and Nifi-Registry services

```bash
$ docker-compose -f docker/docker-compose.nifi.yml up -d
```

- After the Nifi docker-compose is up you will need to get the generated Username and Password to access Nifi.

    - For Username

    ```bash
    $ docker logs docker_nifi_1 | grep "Generated Username"
    ```

    - For Password

    ```bash
    $ docker logs docker_nifi_1 | grep "Generated Password"
    ```

- After you login Apache Nifi you have to import the epic-network-template.xml that is located in the "templates" directory and place it on the workspace.

![Nifi Flow](https://i.ibb.co/7jTdYVZ/workspace.png)

- If you see EPIC-NETWORK group everything is correct.

![Nifi Flow](https://i.ibb.co/hyjDCY3/nififlow.png)

- All processors are located within EPIC-NETWORK

## Cassandra Setup

- Start cassandra-node service

```bash
$ docker-compose -f docker/docker-compose.cassandra.yml up -d
```

- Now we have to create the keyspace and the table "cars" that is going to store the telemetry data

- Accessing Cassandra container

```bash
$ docker exec -it cassandra-node bash
```

- Entering Cassandra terminal

```bash
cqlsh
```

- Create "epicnet" keyspace

```bash
create keyspace epicnet with replication = {'class': 'SimpleStrategy', 'replication_factor': 1};
```

- Create "cars" table

```bash
CREATE TABLE cars (   id UUID PRIMARY KEY,   year int,   make text,   model text,   category text,   slug text );
```

- Now Cassandra database is ready to be used.


## Node-RED Setup

- Start node-red service

```bash
$ docker-compose -f docker/docker-compose.nodered.yml up -d
```

- Open Node-RED in the localhost

```bash
localhost:1880
```

- Necessary installations:

    - Install node-red-contrib-kafka-manager 
    - Install node-red-dashboard

- These installations are intended to use the latest Node-RED node packages.

- Then import the flow.json file that is located in the templates directory

![Node-RED Flow](https://i.ibb.co/2N70n9C/nodered.png)

- If you see this everything is correct!

## Let's run the system

- With all the producers running, we just need to start our Nifi Flow for the system to work

![Node-RED Flow](https://i.ibb.co/yFWCK1c/running.png)

- If you got to

```bash
localhost:1880/ui
```

- You will see the custom EPIC-NETWORK dashboard for the alerts and vehicles histogram

![Nifi Flow](https://i.ibb.co/Cn2qnXj/node-red-dashboard.png)


## Tech Stack

- Docker, Apache Kafka, Apache Nifi, Node-RED, Cassandra, Python




## Authors


- Tomás Navarro
- [@tomasnavb](https://www.github.com/tomasnavb)

