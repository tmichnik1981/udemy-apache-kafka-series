# Apache Kafka Series

#### Kafka Ecosystem

- Topics (Landoop): View the topics content
- Schema UI (Landoop): Explore the Schema registry
- Connect UI (Landoop): Create and monitor Connect tasks
- Kafka Manager (Yahoo): Overall Kafka Cluster Management
- Burrow (LinkedIn): Kafka Consumer Lag Checking
- Exhibitor (Netflix): Zookeeper Configuration, Monitoring, Backup
- Kafka Monitor (LinkedIn): Cluster health monitoring
- Kafka Tools (LinkedIn): Broker and topics administration tasks simplified
- Kafkat (Airbnb): More broker and topics 
- JMX Dump: Dump JMX metrics from Brokers
- Control Centre / Auto Data Balancer / Replicator (Confluent): Paid tools

#### Kafka Core

##### Topics and partitions

- Stream of data
- Similar to a table in a db 
- As many topics as you want
- identified by its name
- Topics are split in partitions
- Each partition is ordered
- Each message within a partition gets an incremental id, called offset
- Offset only have a meaning for a specific partition
- Order is guaranteed only within a partition (not across partitions)
- Data is kept only for a limited time (2 weeks by default)
- Once the data is written to a partition, it can't be changed 
- Data is assigned randomly to a partition unless a key is provided 
- You can have as many partitions per topic as you want

##### Brokers

- A Kafka cluster is composed of multiple brokers (servers)
- Each broker is identified with its ID (int)
- Each broker contains certain topic partitions
- After connecting to any broker (called a bootstrap broker), you will be connected to the entire cluster
- A good number to get started is 3 brokers, but some big clusters have over 100 brokers

##### Topic replication factor

- Topics should have a replication factor > 1 (usually 2 - 3)
- When a broker is down, another broker can serve the data
- At any time only 1 vroker can be a leader for a given partition
- Only that leader can receive and serve data for a partition
- The other brokers will synchronize the data

##### Producers

- Producers write data to topics
- They have to specify the topic name and one broker to connect to, and Kafka  will automatically take care of routing the data to the right brokers
- Acknowledgments:
  -  Acks=0: Producer wont wait for ack (possible data loss)
  - Acks=1: Producer will wait for leader ack (limited data loss)
  - Acks=all: Leader + replicas ack (no data loss)
- Message keys:
  - Producers can choose to send a key with the message
  - If a key is sent, all messages for that key will always go to the same partition
  - This enables ordering

##### Consumers

- Consumers read data from a topic
- They only have to specify the topic name and one broker to connect to, and Kafka will automatically take care of pulling the data from the right brokers
- Data is read in orderr fro each partitions

##### Consumer Groups

- Consumers read data in consumer groups
- Each consumer within a group reads form exclusive partitions
- You cannot have more consumers than partitions (otherwise some will be inactive)

##### Consumer Offsets

- Kafka stores the offsets at which a consumer group has been reading
- The offsets commit live in a Kafka topic named "__consumer_offsets"
- When a consumer has processed data received some Kafka it should be commiting the offsets
- If a consumer process dies, it will be able to read back from where it left off thanks to consumer offsets

##### Zookeper

- Zookeeper manages brokers (keeps a list of them)
- Zookeeper helps in performing leader election for partitions
- Zookeeper sends notifications to Kafka in case of changes (e.g. new topic, broker dies, broker comes up, delete topics, etc)
- Kafka can't work wthout Zookeeper
- Zookeeper usually operates in an odd quorum (cluster) of servers (3, 5,7)
- Zookeeper has a leader, the rest of the servers are followers

##### Kafka Guarantees

- Messages are appended to a topic-partition in the order they are sent
- Consumers read messages in the order stored in a topic-partition
- With a replication factor of N, producers and consumers can tolerate up to N-1 brokers being down
- This is why a replication factor of 3 is a good idea:
  - Allows for one broker to be taken down for maintenance
  - Allows for another broker to be taken down unexpectedly
- As long as th enumber of partitions remains constant for a topic (no new partitions), the same key will always go to the same partition

##### Delivery semantics for consumers

- Consumers choose when to commit offsets
- At most once: offsets are committed as soon as the message is received. If the processing goes wrong the mesage will be lost (it wont be read again)
- At least once: offsets are commited after the message is processed. If the processing goes wrong, the message will be read again. This can result in duplicate processing of messages. Make sure your processing is idempotent (processing again the messages won't impect your systems)
- Exactly once: Very difficult to achieve/needs strong engineering
- Botton line: most often you should use at least once processing and ensure your transformations / procvessing are idempotent

#### Starting Kafka

```shell
# Docker for Mac >= 1.12, Linux, Docker for Windows 10
docker run --rm -it \
           -p 2181:2181 -p 3030:3030 -p 8081:8081 \
           -p 8082:8082 -p 8083:8083 -p 9092:9092 \
           -e ADV_HOST=127.0.0.1 \
           landoop/fast-data-dev

# Docker toolbox
docker run --rm -it \
          -p 2181:2181 -p 3030:3030 -p 8081:8081 \
          -p 8082:8082 -p 8083:8083 -p 9092:9092 \
          -e ADV_HOST=192.168.99.100 \
          landoop/fast-data-dev

# Kafka command lines tools
docker run --rm -it --net=host landoop/fast-data-dev bash
```

- [Landoop Kafka Web GUI](http://127.0.0.1:3030/)

#### Hands-on Practice

##### Topic: create, list , delete

```shell
# list of commands
kafka-topics
kafka-topics --zookeeper 127.0.0.1:2181 --create --topic first_topic --partitions 3 --replication-factor 1
```

