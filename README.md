## Broker
cluster có chứa rất nhiều broker ( servers )
được định danh bằng id
mỗi broker chứa certain topic partitons
sau khi kết nối tới broker ( được gọi là bootstrap broker ) bạn sẽ kết nối tới toàn bộ cluster

### Example of topic-A với 3 partitions với 3 broker

![Alt text](image/broker_with_3_partitions.PNG?raw=true "Title")

### Example of topic-B with 2 partitions

![Alt text](image/topic_b_with_partitions.PNG?raw=true "Title")

## Topic replication factor
Topics nên có replication factor lớn hơn 1 thường sẽ là 2 hoặc 3
Nếu broker có down, thì con broker khác sẽ lên serve the data

### ví dụ về 3 broker 2 replication factor

topic-A có 2 broker: broker 101 và broker 102

partition 0 được assigned vào broker 101

partition 1 được assigned vào broker 102

==> quá trình hình thành Replication Factor

partition 0 topic-A đang ở Broker 101 thì sẽ được replicas ở  Broker 102

partition 1 topic-A đang ở Broker 102 thì sẽ được replicas ở Broker 103
![Alt text](image/broker_with_2_repliation_factor.PNG?raw=true "Title")

Trong trường hợp chúng ta bị mất Broker 102 -> Broker 101 và Broker 103 vẫn serve the data

## Concept of Leader à Partition
Tại mội thời điểm nhất định một broker có thể làm leader ở một số partitions nhất định

Chỉ leader có thể nhận và serve data cho partiton

Các brokers còn lại sẽ synchronize the data

 ## Producers

Producers write data cho thằng topic ( which is made of partitons)

Producers tự động biết cái broker và partition nào để write vào bạn không cần chỉ định nó

Trong trường hợp Broker failures, thì producers sẽ tự động recover

![Alt text](image/produces.PNG?raw=true "Title")

Producers có thể chọn nhận acknowledgment of data writes 
acknowledgement được coi là đồng nghĩa  cho sự xác nhận

acks=0 producers không đợi acknowledgment ( có thể khiến data mất ) gửi cho tới broker. broker đang processing thì chết ==>  data mất ==> dangerous

acks=1 producer sẽ đợi cho leader acknowledgment ( giới hạn việc data mất )

acks=all leader + replicas acknowledgment ( không có data mất )

## Producers: message keys

Producers có thể chọn key cùng với message ( string, number, etc, ... )

Nếu key=null thì data sẽ send theo kiểu round robin ( broker 101 rồi broker 102 rồi tiếp theo broker 1030,.. rồi quay ngược lại )

Nếu key được sent thì tất cả message cho điều đó key sẽ luôn luôn đi tới cùng partition

a key về cơ bản được gửi nếu bạn cần thông tin message được ordering cho một vài field nhất định ( ex: truck_id )

## Consumers

consumers sẽ đọc data từ topic ( được định danh bởi tên )

consumers biết broker nào để đọc từ nó

broker failures, consumers biết làm sao để recover lại

data là được đọc theo thứ tự trong mỗi partitions

![Alt text](image/consumer.PNG?raw=true "Title")

Nó sẽ đọc theo partition lần lượt theo order, tức là nó không thể đọc offset 3 mà chưa đọc offset 2 trong partiton bất kỳ, và nó đọc partition 1 cũng đọc 1 ít ở partiton 2, 1 ít ở partiton 3 đồng thời

## Consumer Groups
consumers đọc data ở trong consumer groups
mỗi consumer trong group đọc từ exclusive partitons

![Alt text](image/consumer_groups.PNG?raw=true "Title")

nếu bạn nhiều consumers hơn partitions, một vài consumers sẽ là inactive

![Alt text](image/consumer_groups_2.PNG?raw=true "Title")

## Consumer Offsets

kafka lưu offsets cái mà consumer group đã được đọc

offsets committed live trong kaffka với tên của topic là __consumer_offsets


Khi mà consumer trong group đã xử lý dữ liệu nhân về từ kafka, nó nên được committing the offsets

Nếu consumer die, nó sẽ có thể đọc lại được từ khi nó rời đi và điều đó phải nhờ tới committed consumer offsets

![Alt text](image/consumer_offsets.PNG?raw=true "Title")

Như bạn có thể hiểu được như là: consumer đọc được và xử lý xong thằng 4262 nó sẽ commit lại. Nêu consumer nó có die thì khi sống dậy nó sẽ tiếp tục đọc next after thằng 4262 là thằng 4263

## Delivery semantics for consumer 

consumer có 3 cách chọn khi commits the offsets

### at most once
offsets sẽ luôn được commit ngay sau khi message được nhận

Nếu quá trình process diện ra sai theo bất kỳ một lý do nào đó thì message đó sẽ bị mất ( bởi vì nó sẽ không được đọc lại )

### at least once
offset sẽ được commit ngay sau message xử lý xong
Nếu quá trình xử lý có bị lỗi thì message sẽ được đọc lại

### extract once

## Kafka broker discovery

tất cả kafka broker đều có thể gọi là bootstrap server

Nghĩa là nếu bạn chỉ cần kết nối tới 1 broker bạn kết nối tới toàn bộ cluster

mỗi broker đều hiểu tất cả brokers, topics và partitons ( metadata )

![Alt text](image/broker_discovery.PNG?raw=true "Title")


## Zookeeper

zookeeper quản lý các brokers ( giữ list của chúng )

zookeeper giúp cho việc perfoming leader cho mỗi partitons

zookeeper gửi thông báo cho kafka trong trường hợp có gì thay đổi

Ví dụ: nếu topic, broker chết, broker sống lại, xóa topic thì zookepper phải gửi đên kafka thông báo

kafka sẽ không thể hoạt động nếu thiếu zookeeper

zookeeper theo design sẽ được khởi tạo với số lẻ server nhưu 3 5 7

zookeeper sẽ có 1 leader xử lý việc write , còn phần còn lại thì sẽ là follower xử lý việc read

phiên bản v 0.10 thì zookeeper sẽ không store consumer offset nữa

## kafka guarantees

message được chèn vào topic-partition theo thứ tự mà chúng đã gửi

consumer sẽ đọc message theo thứ tự mà chúng stored in topic-partition

với replication factor of N, producers và consumers có thể chịu đựng được đến N-1 brokers beeing down

Lý do tại replication factor là 3 thì là ý kiến hay:
cho phép 1 broker bị down vì lý do maintenance
cho phép 1 broker bị down vì lý do unexpectedly