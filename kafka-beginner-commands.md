# Kafka stuff


* Command to verify network ports:

```
➜ nc -v -z localhost 2181
Connection to localhost port 2181 [tcp/eforward] succeeded!
```

* Properties files:
    * Kafka: `server.properties`
    * Zookeeper: `zookeeper.properties`
* Create topic:

```
kafka-topics.sh --bootstrap-server 127.0.0.1:9092 --topic first_topic --create --partitions 3 --replication-factor 1
```

* List topic:

```
kafka-topics.sh --bootstrap-server 127.0.0.1:9092 --topic first_topic --create --partitions 3 --replication-factor 1
```

* Describe a topic:

```
kafka-topics.sh --bootstrap-server 127.0.0.1:9092 --topic first_topic --describe

Topic: first_topic	TopicId: fBTBdhOsQomnLt6s1wDbtw	PartitionCount: 3	ReplicationFactor: 1	Configs: segment.bytes=1073741824
	Topic: first_topic	Partition: 0	Leader: 0	Replicas: 0	Isr: 0
	Topic: first_topic	Partition: 1	Leader: 0	Replicas: 0	Isr: 0
	Topic: first_topic	Partition: 2	Leader: 0	Replicas: 0	Isr: 0
```

* Delete a topic

```
kafka-topics.sh --bootstrap-server localhost:9092 --topic second_topic --delete
```

* Produce to a topic:

```
kafka-console-producer.sh --broker-list localhost:9092 --topic first_topic
>Henlo
>How are you
>Where are you
CTRL+C
```

* Produce to a non\-existent topic:

```
kafka-console-producer.sh --broker-list localhost:9092 --topic second_new_topic
>this is another topic
[2022-01-18 10:47:42,974] WARN [Producer clientId=console-producer] Error while fetching metadata with correlation id 3 : {second_new_topic=LEADER_NOT_AVAILABLE} (org.apache.kafka.clients.NetworkClient)
[2022-01-18 10:47:43,096] WARN [Producer clientId=console-producer] Error while fetching metadata with correlation id 4 : {second_new_topic=LEADER_NOT_AVAILABLE} (org.apache.kafka.clients.NetworkClient)
[2022-01-18 10:47:43,202] WARN [Producer clientId=console-producer] Error while fetching metadata with correlation id 5 : {second_new_topic=LEADER_NOT_AVAILABLE} (org.apache.kafka.clients.NetworkClient)
>yet another topic
>henlo
>how are you?
>こんにちは
>^C%
```

* Consume in real time:

```
kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic second_new_topic
```

* Consume from beginning from a topic:

```
kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic second_new_topic --from-beginning
this is another topic
henlo
皆さんこんにちは！
yet another topic
how are you?
こんにちは
僕はサウラブと申します。インド人です。
```

* Consumers in groups:
    * Run 3 consumer groups while producing from a single producer:

```
kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic second_new_topic --group my-app```
    * Now turn of all consumers and continue producing messages:

```
kafka-console-producer.sh --broker-list localhost:9092 --topic second_new_topic
```
    * Again, start a new consumer group:

```
kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic second_new_topic --group my-app
```

Observe that all the messages appended after closing the consumer groups will appear in the newly created consumer group. But not the ones from beginning.

* Resetting offsets:

```
kafka-consumer-groups.sh --bootstrap-server localhost:9092 --group my-app --topic second_new_topic --reset-offsets --to-earliest --execute

GROUP                          TOPIC                          PARTITION  NEW-OFFSET
my-app                         second_new_topic               0          0
my-app                         second_new_topic               1          0
my-app                         second_new_topic               2          0
```

* Verify current lag:

```
kafka-consumer-groups.sh --bootstrap-server localhost:9092 --describe --group my-app

Consumer group 'my-app' has no active members.

GROUP           TOPIC            PARTITION  CURRENT-OFFSET  LOG-END-OFFSET  LAG             CONSUMER-ID     HOST            CLIENT-ID
my-app          second_new_topic 0          0               20              20              -               -               -
my-app          second_new_topic 1          0               15              15              -               -               -
my-app          second_new_topic 2          0               10              10              -               -               -
```

* Re\-start the consumer:

```
kafka-console-consumer.sh --bootstrap-server localhost:9092 --group my-app --topic second_new_topic

this is another topic
henlo
皆さんこんにちは！
henlo
tokyo
mumbai
kolkata
green
sky
new messages
hello
hi
this is another test message
yet another topic
how are you?
berlin
london
delhi
hi
howdoyoudo
blue
red
the
is
```

* Verify that the lag is again back to zero:

```
kafka-consumer-groups.sh --bootstrap-server localhost:9092 --describe --group my-app

Consumer group 'my-app' has no active members.

GROUP           TOPIC            PARTITION  CURRENT-OFFSET  LOG-END-OFFSET  LAG             CONSUMER-ID     HOST            CLIENT-ID
my-app          second_new_topic 0          20              20              0               -               -               -
my-app          second_new_topic 1          15              15              0               -               -               -
my-app          second_new_topic 2          10              10              0               -               -               -
```
