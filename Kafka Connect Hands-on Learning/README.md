# Kafka Connect Hands-on Learning

#### Kafka Connect Concepts

##### Kafka Connect and Streams

- Source => Kafka    **Producer API**
- Kafka => Kafka    **Consumer, Producer API**
- Kafka => Sink     **Consumer API**
- Kafka => App     **Consumer API**

##### Architecture Design

1. Sources -ie mongo
2. Workers pull data from sources and sends to
3. Kafka cluster 
4. Streams - agregrations, exchanging data, joins
5. Sinks - databases, ELK, HTFS - place where data could be placed at the end

##### High level

- Source Connectors to get data from Common Data Sources
- Sink Connectors to publish that data in Common Data Stores
- Make it easy for non-experienced dev to quickly get their data reliably into Kafka
- Part of your ETL pipeline
- Scaling made easy from small pipelines to company-wide pipelines
- Re-usable code

##### Concepts

- Kafka Connect Cluster has multiple loaded **Connectors**
  - Each connector is a re-usable piece of code (java jars)
  - Many connectors exist in the open source world, leverage them!
- Connectors + **User Configuration** => **Tasks**
  - A task is linked to a connector configuration
  - A job configuration may spawn multiple tasks
- Tasks are executed by Kafka Connect **Workers** (servers)
  - A worker is a single java process
  - A worker can be standalone or in a cluster

##### Standalone vs Distributed Mode

- Standalone:
  - A single process runs your connectors and tasks
  - Configuration is bundled with your process
  - Very easy to get started with, **useful for development and testing**
  - Not fault tolerant, no scalability, hard to monitor
- Distributed:
  - Multiple workers run your connectors and tasks
  - Configuration is submitted using a REST API 
  - Easy to scale, and fault tolerant (rebalancing in case a worker dies)
  - Useful for **production deployment of connectors**

#### Setup

- Install Docker

- Use file  **docker-compose.yml** for running containers

  ```bash
  # start out Kafka cluster
  docker-compose up kafka-cluster
  
  # We start a hosted tools
  docker run --rm -it --name=kafka-commands -net=host landoop/fast-data-dev bash
  ```

- Landoop UI: 127.0.0.1:3030/kafka-connect-ui

- Kafka Connect Logs

  - In standalone mode - in your terminal
  - In distributed mode
    - at the URL http://127.00.0.1:3030/logs/
    - file: **connect-distributed.log**
  - Look for ERROR in the log, that's a good indicator of what went wrong (an error message will come with the error)

#### Kafka Connect Source

##### Goal

- Read a file and load the content directly into Kafka
- Run in a connector in **standalone mode** (useful for development)

##### Files & Commands 

- Starting cluster:` docker-compose up kafka-cluster`
- kafka-connect-tutorial-sources.sh - instructions
- demo-1:
  - worker.properties - configures worker in a standalone mode
  - file-stream-demo-standalone.properties - setup of the connector
  - demo-file.txt - source file

http://127.0.0.1:3030/



TODO: przerobić kafka-connect-tutorial-sources.sh - instructions



