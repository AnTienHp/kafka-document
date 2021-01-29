## Topic

Topics a particular stream of data

Similar to table in database ( without all constraint )

You can have have as many topic as you want

A topic is identified by its name

Topics are split in partitions

Each partition is ordered

Each message with a partiton gets an incremental id called offset

The first message to partition zero is going have the offset zero next one is offset one, ect, etc ...

Example:
You have a lot of car, and each car has GPS in it, basicly we want to each car to report its GPS position to kafka

you can have topic and you named it is car_gps that contains the position of all car

All car will send a message to kafka every 20 seconds, each message will contains the car_id and car_position


If you have 1000 car, and 1000 car will send message to topic car_topic 

Noted: 
1. offset only have meaning  for a  specific partition
2. order is guaranteed only within a partition
3. data is kept only for a limited time ( default is a one week )
4. Once data is written to a partition. It can't be changed
5. Data is assigned randomly to a partition unless key is provided 

## Broker

A kafka cluster is conposed a multiple brokers ( server )

Each broker is identified with the id ( interger )

Each broker contains certain topic partition
-> each broker has some kind of data, but not all the data

After connection to any broker ( called bootstrap broker ) you will be connected to the entire cluster

### Example of topic-A với 3 partitions với 3 broker

![Alt text](image/broker_with_3_partitions.PNG?raw=true "Title")

Partition number and the broker number there is a no relationship. It could be in whatever order. But as you can see. When you create a topic, kafka will automaticly assign topic and dsitribute to all brokers 

### Example of topic-B with 2 partitions

![Alt text](image/topic_b_with_partitions.PNG?raw=true "Title")

## Topic replication factor

if the machine go does, and everything still work => replication does that for us. 

Topics should have  a replication factor > 1 ( usually 2 and 3 )

This way if a broker is down, another broker still server data

### Topic-A with 2 partitions and replication factor of 2

![Alt text](image/broker_with_2_repliation_factor.PNG?raw=true "Title")

If mean you have two copies of each data

Example we lost Broker 102 -> Broker 101 and 103 still serve the data -> to ensure data wouldn't be lost

## Concept of Leader for a Partition

At any time, only one broker can be a leader for a given partition

Only the leader can receive and serve data for a partition

The other broker will be synchronize the data

Therefor each partition has one leader and multi ISR ( in-sync replicas )

There's a leader, and there a lot of ISR. zookeeper decides who is leaders and replicas


## Producers

1. Producers write data to topics ( which made of partitions )
2. Producers automatically know which broker and partition to write to
3. incase of Broker failures, Producers will automatically recover

![Alt text](image/produces.PNG?raw=true "Title")

Producers can choose to receive acknowledgment of data to write

acknowledgment is a synonym for confirmation

there is 3 kind of ack

acks=0 producers won't wait for ack => possible data loss

acks=1 producer will wait for  leader ack ( limited data loss )

acks=all leader and repllicas ack ( no data loss )

## Producers: message keys

Producers can choose to send a key with the message

if key = null data is sent to round robin

if a key is sent then all messages for that key will always go to same partition


![Alt text](image/message_key.PNG?raw=true "Title")

## Consumers

Consumer read data from topic ( identified by name )

Consumer know which broker to read from

in case of broker failures, consumers know how to recover

Data is read in order within a partitions


![Alt text](image/consumer.PNG?raw=true "Title")

## Consumer Groups

Consumer read data in Consumer Groups

Each consumer with a group reads from  exclusive partitions

![Alt text](image/consumer_groups.PNG?raw=true "Title")

If you have more consumers than partition, som consumer will be an inactive

![Alt text](image/consumer_groups_2.PNG?raw=true "Title")

## Consumer Offsets

kafka stores the offsets at which a consumer group is reading

The offsets comitted live in kafka topic named __consumer_offsets

When a consumer in a group has processed data received from kafka, it committing the offsets

if consumer die, it will be able to alive and read back from where it left

![Alt text](image/consumer_offsets.PNG?raw=true "Title")

## Kafka broker discovery

Each kafka broker is also called "bootstrap server:
-> that means you only need to connect one of them and you will be connected to a entire cluster

Each broker knows about all brokers, topic and partition ( metadta )

![Alt text](image/broker_discovery.PNG?raw=true "Title")


## Zookeeper

Zookeeper manages broker

Zookeeper help in performing leader and RIS for partitions

Zookeeper send notification to kafka about new topic, brokers dies, broker come alive, topic deleted, ...

zookeeper by design operates with odd number of server (1, 3, 5, 7 ...)

Zookeeper has a leader and the rest of server is follower

## kafka guarantees

message are appended to a topic-partition in the order they sent

consumers read message in the order stored in a topic-partition

why is replication factor is 3 is a good idea
1. allows for one broker to be taken down for maintenance
2. allows for another broker to be taken down unexpectedl