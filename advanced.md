acks means acknowledgments

acks = 0 mean no response is requested
if the broker goes offline or exeception happens, will lose data

useful for data where it's okay to potentially lose messages like
1. metric collection
2. log collection

=> nice perfomance

acks = 1 leader acks
leader response is requested, but replication is not guarantee ( happens in the background )
but it's not a prerequisite to receiving a response
if an ack is not received the producer may retry

producer send data to leader and leader response to every write request

if the leader broker goes offline but replicas haven't replicaed the data yet, we have a data loss


acks = all replicas acks

leader + replicas ack requested

added latency and safety

no data loss if enough replicas => necessary setting if you don't want to lose data

acks=all must be used in conjunction with min.insync.replicas
min.insync.replicas an be set at the broker or topic level ( override )

min.insync.replias=2 implies that at least 2 brokers that isr ( including leader ) must response that they have data

that means if you use replication.factor=3, min.insync=2, acks=all, you can
only tolerate i broker going down, otherwise the producer will receive an 
exception on send

then basically your topic won't be working well if more than one
broker is down, for example two brokers

demo:
min.insync.replicas=2
2 broker down only 1 broker is leader is alive
producer send data to leader, by default leader send data to 2 broker 
but 2 broker is down. so, leader will send exception NOT_ENOUGH_REPLICAS




PRODUCER RETRIES

in case of transient failures, developer are expected to handle exceptions,
otherwise the data will be lost

Example of transient failure
NotEnoughReplicasException

Retries was born by that
default to 0 for kafka <= 2.0 => if you version is low, your producer will not retry automatically to send your message
if version of kafka >= 2.1 default is 2^31 -1 mean a lot very high number of times

the retry.backoff.ms setting is by default 100ms
mean that every 100 miliseconds your kafka will retry the send to kafka
is that mean if your version is higher than 2.1. you retry 2^31 -1 and between two retries is 100ms
=> mean forever :v is that right? No. 

The producer won't try the request for ever, it's bounded by a timeout => KIP was born

delivery.timeout.ms = 120000ms == 2 minutes by default

you retries, retries. after 2 minutes. you done. => the data will sent to kafka anymore

=> you looking at the error message in your producer callback

records will be failed if the can't be acknowledged in delivery.timeout.ms
allows you to give you some boundaries around how much you want to retry a message being sent before failing it.


In cases of retries, there is a chance that messages will be sent  out of order ( if a batch has failed to be sent )

if you rely on key-based ordering, that can be an issue

for this, you can set the setting while controls how many product requests
can be made in parallel: max.in.flist.request.per.connection
default:5
set it to 1 you need to ensure ordering ( my impact throughput )

=> idempotent producer was born

IDEMPOTENT PRODUCER

good request: it produces data to kafka. kafka got that data. and commit it. and send backs an acknowledgements
duplicate request; producers send produce request to kafka, kafka got data and commit that. then sends back an ack
but ack never reachs ( sometimes due to network error )

producer will retry, because try is better than zero and because i haven't received an ack i retry produce
and kafka will commit duplicate 

==> idempotent producer

in kafka >= 0.11 you can define a idempotent producer which won't introduce duplicate data on network error or something

idempotent request: producers produce data to kafka => kafka got that data => kafka commit
=> then kafka send ack.
ack never reachs due to network error or something 
=> retry produce has a produce request id
=> kafka using that request produce id kafka will detect duplicate and kafka won't commit twice
=> send back an ack

idempotent producers are greate to guarantee a stable and safe pipeline

idempotent producer is off by default
but you turn it on by set producerProps.put("enable.idempotence", true)

MESSAGE COMPRESSION

producer usually send data that is text-based for example with json data
in this case it is important to apply compression to the producer

compression í enabled at the producer level and doesn't required any configuration
change in the brokers on in the cunsumers

compresstion.type can be 'none' by default gzip lz4 or snappy

