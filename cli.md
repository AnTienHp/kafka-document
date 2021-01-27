# THIS IS A LIST OF COMMAND YOU ALWAYS USED FOR TOPICS 

# to create topic with zookeeper port 2181, 3 partitions 1 replication-factor 1 broker
kafka-topics.sh --zookeeper 127.0.0.1:2181 --topic first_topic --create --partitions 3  --replication-factor 1
# to describe topic first_topic which you created 
kafka-topics.sh --zookeeper 127.0.0.1:2181 --topic first_topic --describe 
# to delete topic you could use command.
# first i create a new topic with name is second_topic with 6 partitions and 1 replication-factor
kafka-topics.sh --zookeeper 127.0.0.1:2181 --topic second_topic --create --partitions 6  --replication-factor 1 
# and i delete topic named second_topic
kafka-topics.sh --zookeeper 127.0.0.1:2181 --topic second_topic --delete
# but command it doesn't actually delete second_topic right away
# i won't get deleted if the setting delete topic enable ( delete.topic.enable = true )
# to show list of topic i used command
kafka-topics.sh --zookeeper 127.0.0.1:2181  --list
# for more command you could use command to show all
kafka-topic.sh

# THIS IS A COMMAND YOU ALWAYS USED FOR PRODUCER CLI

# command use to start console producer with broker_list on topic first_topic
kafka-console-producer.sh --broker-list 127.0.0.1:9092 --topic first_topic --producer-property acks=all


# THIS IS COMMAND YOU ALWAYS USED FOR CONSUMER CLI

# command to start console consumer message will stream
kafka-console-consumer.sh --bootstrap-server 127.0.0.1:9092 -topic first_topic
#note kafka-console-consumer uses a random group id if don't specify

# if you want to load message from to start you send from to producer
kafka-console-consumer.sh --bootstrap-server 127.0.0.1:9092 -topic first_topic --from-beginning

kafka-console-consumer.sh --bootstrap-server 127.0.0.1:9092 -topic first_topic --group my-first-application
kafka-console-consumer.sh --bootstrap-server 127.0.0.1:9092 -topic first_topic --group my-second-application --from-beginning


# for consumer group

# kafka-consumer-group to list all consumer group, describe a consumer group, delete consumer group info, or reset consumer group offsets
kafka-consumer-groups.sh --bootstrap-server localhost:9092 --describe --group my-second-application
kafka-consumer-groups.sh --bootstrap-server localhost:9092 --describe --group my-first-application