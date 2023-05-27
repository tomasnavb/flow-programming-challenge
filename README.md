
# Flow Programming Challenge by Tomás Navarro




## Description 🧾

This is my personal project made to participate in the "Flow Programming Challenge" from Epic IO.\
The challenge consists of building a system based on 4 containerized software components (Apache Kafka, Apache NiFi, Node-RED, and a producer). This system is responsible for processing vehicle-related data, transforming messages, and storing them formatted in a database. It also displays alerts on a Node-RED dashboard based on vehicles that meet certain criteria. The programs communicate with each other through a Docker network named 'epic-net'. \
In this document I'm going to explain how to deploy the project.


## Deployment ⚙️


- Follow the next steps to succesfully deploy the project

## Infra Setup 💻

- You will need [Docker](https://docs.docker.com/install/) and [Docker Compose](https://docs.docker.com/compose/) installed in your computer.

- You simply need to create a Docker network called `epic-net` to enable communication between the services.

```bash
$ docker network create epic-net
```

## Producers Setup 🔧

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

## Nifi Setup 🔧

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

![Nifi Flow](https://i.ibb.co/fnvcWyM/principal.png)

- If you see EPIC-NETWORK group everything is correct.

![Nifi Flow](https://i.ibb.co/hyjDCY3/nififlow.png)

- All processors are located within EPIC-NETWORK

- Now set the database password in PUT CASSANDRA processor. This is required because Apache Nifi doesn't save passwords in templates configuration.

```bash
User: admin
Password: admin
```

- Last step is enable CassandraSessionProvider in the EPIC-NETWORK configuration.

![Nifi Flow](https://i.ibb.co/GJqgJYb/enable.jpg)

![Nifi Flow](https://i.ibb.co/8dJTv8R/ok.jpg)


## Cassandra Setup 🔧

In this project I will use a Cassandra database because:

- Native Integration: Apache NiFi has a specific processor called "PutCassandraRecord" that allows writing data directly into a Cassandra database. This native integration simplifies the workflow and configuration when using both systems together

- Scalability and Performance: Both Apache NiFi and Cassandra are designed to be highly scalable and handle large volumes of data. 

- Fault Tolerance and High Availability: Both Apache NiFi and Cassandra are designed to be fault-tolerant and ensure data availability. 

- Real-time Data Streaming Compatibility: Apache NiFi is known for its real-time data processing capabilities. When combined with Cassandra, you can create real-time data processing systems that efficiently store and query data.

- Start cassandra-node service

```bash
$ docker-compose -f docker/docker-compose.cassandra.yml up -d
```

- Now we have to create the 'epicnet' keyspace and the table 'cars' that is going to store the telemetry data

- Accessing Cassandra container

```bash
$ docker exec -it cassandra-node bash
```

- Entering Cassandra terminal

```bash
cqlsh
```

- Create 'epicnet' keyspace

```bash
create keyspace epicnet with replication = {'class': 'SimpleStrategy', 'replication_factor': 1};
```

- Create 'cars' table

```bash
CREATE TABLE cars (   id UUID PRIMARY KEY,   year int,   make text,   model text,   category text,   slug text );
```

- Now Cassandra database is ready to be used.


## Node-RED Setup 🔧

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

## Let's run the system ▶️

- With all the producers running, we just need to start our Nifi Flow for the system to work

![Node-RED Flow](https://i.ibb.co/yVxZqWS/play.jpg)

- If you got to

```bash
localhost:1880/ui
```

- You will see the custom EPIC-NETWORK dashboard for the alerts and vehicles histogram

![Nifi Flow](https://i.ibb.co/Cn2qnXj/node-red-dashboard.png)


## Tech Stack

- Docker, Apache Kafka, Apache Nifi, Node-RED, Cassandra, Python




## Authors


- Tomás Navarro - [@tomasnavb](https://www.github.com/tomasnavb)

