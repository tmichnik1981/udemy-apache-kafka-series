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
- As long as the number of partitions remains constant for a topic (no new partitions), the same key will always go to the same partition

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

# list all topics
kafka-topics --zookeeper 127.0.0.1:2181  --list

# creating a new topic 
kafka-topics --zookeeper 127.0.0.1:2181 --create --topic first_topic --partitions 3 --replication-factor 1

kafka-topics --zookeeper 127.0.0.1:2181 --create --topic second_topic --partitions 3 --replication-factor 1

# deleting a topic
kafka-topics --zookeeper 127.0.0.1:2181 --topic second_topic --delete

# displaying information about a topic
kafka-topics --zookeeper 127.0.0.1:2181 --describe --topic first_topic
```

##### Publishing to a topic

```shell
kafka-console-producer --broker-list 127.0.0.1:9092 --topic first_topic
```

##### Consuming data from topic

```shell
# consumes only new messages
kafka-console-consumer --bootstrap-server 127.0.0.1:9092 --topic first_topic

# consumes all messages
kafka-console-consumer --bootstrap-server 127.0.0.1:9092 --topic first_topic --from-beginning

# consumes all messages as group: mygroup1  and commit offset
kafka-console-consumer --bootstrap-server 127.0.0.1:9092 --topic first_topic --consumer-property group.id=mygroup1 --from-beginning
```

##### Code Examples

- [Kafka Producer and Consumer Examples](https://github.com/apache/kafka/tree/trunk/examples/src/main/java/kafka/examples)
- [Spark Streaming + Kafka Integration Guide](https://spark.apache.org/docs/2.1.0/streaming-kafka-integration.html)
- [Spark Kafka Writer](https://github.com/BenFradet/spark-kafka-writer)
- [Spark Streaming Word Cound Demo](https://github.com/apache/spark/blob/master/examples/src/main/java/org/apache/spark/examples/streaming/JavaDirectKafkaWordCount.java)
- [Akka Streams Introduction](https://doc.akka.io/docs/akka/current/stream/stream-introduction.html?language=java)
- [Akka Streams Kafka Docs](https://doc.akka.io/docs/akka-stream-kafka/current/home.html)
- [Akka Streams Kafka Scala Examples](https://github.com/makersu/reactive-kafka-scala-example)
- [Scala Kafka Client Example](https://github.com/cakesolutions/scala-kafka-client)

##### Apache NiFi to Apache Kafka

- apache NiFi is really useful for buildingdata pipelines using drag and drops with little to no programming, all driven by configuration
- NiFi and Kafka go along well together, allowing you to easily do: Kafka =>Any Source or Any Source => Kafka
- Apache NiFi can be an altrenative to Kafka Connect framework

```shell
# Short instructions for Mac / Linux
# download NiFi at: https://nifi.apache.org/download.html
# Terminal 1:
bin/nifi.sh run
# install docker for mac / docker for linux
# Terminal 2:
docker run -it --rm -p 2181:2181 -p 3030:3030 -p 8081:8081 -p 8082:8082 -p 8083:8083 -p 9092:9092 -e ADV_HOST=127.0.0.1 -e RUNTESTS=0 landoop/fast-data-dev
# Terminal 3:
docker run -it --net=host landoop/fast-data-dev bash
kafka-topics --create --topic nifi-topic --zookeeper 127.0.0.1:2181 --partitions 3 --replication-factor 1
kafka-console-consumer --topic nifi-topic --bootstrap-server 127.0.0.1:9092
```

##### Spring Kafka

- [Spring for Kafka](https://projects.spring.io/spring-kafka/)
- [Pipelines with Spring](https://spring.io/blog/2015/04/15/using-apache-kafka-for-integration-and-data-processing-pipelines-with-spring)
- [Spring Kafka github](https://github.com/spring-projects/spring-kafka)
- [Spring Kafka - integration](https://github.com/spring-projects/spring-integration-kafka)

#### Advanced Topic configuration

- [Broker configs](https://kafka.apache.org/documentation/#brokerconfigs)

```shell
# A string that is either "delete" or "compact". This string designates the retention policy to # use on old log segments. The default policy ("delete") will discard old segments when their # retention time or size limit has been reached. The "compact" setting will enable log compaction # on the topic.

# setting cleanup.policy=compact
kafka-topics --create --topic test_cleanup --zookeeper 127.0.0.1:2181 --config cleanup.policy=compact --partition 3 --replication-factor 1

# updating cleanup.policy
kafka-topics --alter --topic test_cleanup --zookeeper 127.0.0.1:2181 --config cleanup.policy=delete


```

##### Partitions Count, Replication Factor

- [How to choose the number of topics/partitions in Kafka](https://www.confluent.io/blog/how-to-choose-the-number-of-topicspartitions-in-a-kafka-cluster/)
- It is best to get the parameters right the the first time
  - If the Partitions Count increases during a topic lifecycle, you will break your keys ordering guarantees
  - If the Replication Factor increases during a topic lifecycle, you put more pressure on your cluster, which can lead to unexpected performance decrease

###### Partitions Count

- Roughly, each partition can get a throughput of 10MB / sec
- More partitions implies:
  - Better parallelism, better throughput
  - BUT more files opened on your system
  - BUT if a broker fails (unclean shutdown), lots of concurrent leader elections
  - BUT added latency to replicate (in the order of milliseconds)
- Guidelines:
  - Keep the number of partition per brokers between 2000 and 4000
  - Keep the number of partitions in the cluster to less than 20,000
  - Partitions per topic = (1 to 2) x (# of brokers), max 10 partitions
  - Example: in a 3 brokers setup, 3 or 6 partitions is a good number to start with

###### Replication Factor

- Should be at least 2, maximum of 3
- The higher the replication factor:
  - Better resilience of your system (N-1 brokers can fail)
  - BUT longer replication (higher latency is acks=all)
  - BUT more disk space on your system (50% more if RF is 3 instead of 2)
- Guidelines:
  - Set it to 3 (you must have at least 3 brokers for that)
  - if replication performance is an issue get a better broker instead of less RF

##### Partitions and Segments

- Topics are made of partitions (we already know that)
- Partitions are made of... segments (files)!
-  Only one segment is ACTIVE (the one data is being written to)
- Two segment settings:
  - <u>log.segment.bytes:</u> the max size of a single segment in bytes
  - <u>log.segment.ms:</u> the time Kafka will wait before committing the segment if not full
- Segments come with two indexes (files):
  - An offset to position index: allows Kafka where to read to find a message
  - A timestamp to offset index: allows Kafka to find messages with a timestamp
- Therefore, Kafka knows where to find data in a constant time!
- A smaller **log.segment.bytes** (size, default: 1GB) means:
  - More segments per partitions
  - Log Compaction happens more often
  - BUT Kafka has to keep more files opened (Error: Too many open files)
-  A smaller log.segment.mc (time, default 1 week) means:
  - You set a max frequency for log compaction (more frequent triggers) 
  - Maybe you want daily compaction instead of weekly? 

##### Log Cleanup Policies

- Many Kafka clusters make data expire, according to a policy
- That concept is called log cleanup

###### Policy 1: log.cleanup.policy=delete (Kafka default for all user topics)

- Delete based on age of data (default is a week)
- Delete based on max size of log (default is -1 == infinite)

###### Policy 2: log.cleanup.policy=compact (Kafka default for topic __consumer_offsets)

- Delete based on keys of your messages
- Will delete old duplicate keys after the active segment is committed
- Infinite time and space retention

Log Cleanup: Why and When?

- Deleting data from Kafka allows you to:
  - Control the size of the data on the disk, delete obsolete data
  - Overall: Limit maintenance work on the Kafka Cluster
- How often does log cleanup happen?
  - Log cleanup happens on your partition segments!
  - Smaller / More segments means that log cleanup will happen more often!
  - Log cleanup shouldn't happen too often => takes CPU and RAM resources
  - The cleaner checks for work every 15 seconds (**log.cleaner.backoff.ms**)

##### Log Cleanup Policy: Delete

- log.retention.hours:
  - number of hours to keep data for (default is 168 - one week)
  - Higher number means more more disk space
  - Lower number means that less data is retained (your consumers may need to replay more data than less)
- log.retention.bytes:
  - Max size in Bytes for each partition (default is -1 - infinite)
  - Useful to keep the size of a log under a threshold

##### Log Cleanup Policy: Compact

- [Getting Started with Apache Kafka for the Baffled part 2](http://www.shayne.me/blog/2015/2015-06-25-everything-about-kafka-part-2/)


- Log compaction ensures that your log contains at least the last known value for a pecific key within a partition
- Very useful if we just require a SNAPSHOT instead of full history
- The idea is that we only keep the latest "update" for a key in our log
- Log compaction removes old keys if newer ones are available
- **Any consumer that is reading form the head of a log will still see all the messages sent to the topic**
- Ordering of messages it kept, log compaction only removes some messages, but does not  reorder them
- The offset of a message is immutable. Offsets are just skipped if a message is missing
- Deleted records can still be seen by consumers for a period of delete.retention.ms (default is 24 h)
- Log compaction does not prevent ypu form pushing duplicate data to kafka
  - De-duplication is done after a segment is committed
  - Your consumers will still read from head as soon as the data arrives
  - **Only new consumers wont see deleted (compacted) records**
- It does not prevent you form reading duplicate data form Kafka
- Log compaction can fail from time to time

##### Log Compression

- [Kafka Perormance testing](https://cwiki.apache.org/confluence/display/KAFKA/Performance+testing)
- Topics can be compressed using **compression.type**  setting.
- Options are **gzip**, **snappy**, **lz4**,  **uncompressed**, **producer**
- If you need compression, ideally you keep default as **producer**
  - The producer will perform the compression on its side
  - The broker will take the data as is => Saves CPU resources on the broker
- If compression is enabled, make sure you're sending batches of data
- Data will be uncompressed by the consumer!
- Compression only makes sense if you're sending non-binary data (compress:JSON, test... | don't compress: Parquet, Protobuf, Avro) 

##### Other advanced configurations

- [blog post about min.insync.replicas](https://logallthethings.com/2016/07/11/min-insync-replicas-what-does-it-actually-do/)


- **max.messages.bytes** (default is 1 MB): if your messages get bigger than 1MB, increase this parameter on the topic and your consumers buffer
- **min.isync.replicas** (default is 1): if using acks=all, specify how many brokers need to acknowledge the write
- **unclean.leader.election** (danger!! - default true): id set to true, it will allow replicas who are in sync to become leader as a last resort if all ISRs are offline. This can lead to data loss. If set to false, the topic will go offline until the ISRs come back up.