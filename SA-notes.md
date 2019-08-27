[SNS](https://github.com/Kinnoshachi/notes/blob/master/SA-notes.md#sns)
## ___________________________

# SQS

## ACG NOTES
* 256 kb text p/sqs
* standard queue and FIFO Queues
* standard is best effort ordering and could deliver sqs more than once
* FIFO ordered no duplicates 300 tx/s
* pull based
* 1m -14d, 4d default retention
## _____________________


#### SQS works as a buffer between application prcessing components, guarantees at least one delivery, supporting multiple readers and writers. 
#### System design should be idempotent; such that there is no negatove effect if it processess message twice. 
#### FIFO is not guaranteed. Sequencing information can be put in each message such that they can be reordered when retrievd from the queue if ordering is necessary.
#### SQS lifecycle
![i2](https://learning.oreilly.com/library/view/aws-certified-solutions/9781119138556/images/ec08f001.jpg)

### Delay and lifecycle timeout
####    Delay queues are similiar to visibility timeouts, and allow you to postpone the delivery of new messages in a queue for a specific number of seconds
#### use ```CreateQueue``` and set the ```DelaySeconds``` attribute to any value between 0 and 900 (15 minutes). 
#### turn an existing queue into a delay queue by using ```SetQueueAttributes``` to set the queue’s `DelaySeconds` attribute. 
#### default value for    `DelaySeconds` is 0
#### delay queue hides a message when it is first added to the queue, whereas a visibility timeout hides a message only after that message is retrieved from the queue. When visible it is *in flight*
![i3](https://learning.oreilly.com/library/view/aws-certified-solutions/9781119138556/images/ec08f002.jpg)

* up to 120,000 messages in flight at any given time. 
* SQS supports up to 12 hours’ maximum visibility timeout
* accessed through HTTP request-response
* request-response takes a bit less than 20ms from EC2; @50 api req p/sec. More threads == more throughput via horizontal scaling.

### Queue Operations, Unique IDs, and Metadata
~~~~
CreateQueue
ListQueues
DeleteQueue
SendMessage
SendMessageBatch
ReceiveMessage
DeleteMessage
DeleteMessageBatch
PurgeQueue
ChangeMessageVisibility
ChangeMessageVisibilityBatch
SetQueueAttributes
GetQueueAttributes
GetQueueUrl
ListDeadLetterSourceQueues
AddPermission
RemovePermission
~~~~

#### each message has a globally unique ID. response includes a receipt handle, which you must provide when deleting the message.max len 1024 char

### QUEUE AND MESSAGE IDENTIFIERS

#### 3 identifiers: 
1. queue URLs
1. message IDs
1. receipt handles
#### create a queue > name unique within your queues > aws assigns Queue URL > to perform action on queue must provide its name
####  SQS assigns each message a unique ID that it returns in the `SendMessage` response. max len 100 char


### Message Attributes
#### allow you to provide structured metadata items (such as timestamps, geospatial data, signatures, and identifiers) optional, seperate from but sent with message body such that the reciever can determine how to process before evaluating the mssg body. Up to 10 attribute per mssg
#### specify message attributes via:
- AWS Management Console
- SDKs 
- query API.

### Long Polling
#### When application queries SQS queue for messages, it calls the function `ReceiveMessage` however this burns cpu cycles and ties up a thread if caals are looped rather than periodic. Therefore use longpolling pattern. If no messages in queue long polling, executes WaitTimeSeconds argument to ReceiveMessage of up to 20 seconds

### DEAD LETTER QUEUES
#### helps to sideline and isolate the unsuccessfully processed messages. Dead letter queue sends/recieves mssgs like any other service. Create DLQ via SQS API or console
### IAM
#### following illustrates developer with AWS account number 111122223333 the `SendMessage` permission for the queue named 444455556666/queue1 in the US East (N. Virginia) region
````json
{"Version": "2012&#x02013;10–17",
    "Id": "Queue1_Policy_UUID","Statement": 
    [
        {"Sid":"Queue1_SendMessage","Effect": "Allow","Principal": {
            "AWS": "111122223333"
            },
            "Action":"sqs:SendMessage",
            "Resource": "arn:aws:sqs:us-east-1:444455556666:queue1"}]}
````

#### Amazon SQS Access Control allows you to assign policies to queues that grant specific interactions to other accounts without that account having to assume IAM roles from your account.
#### Instead of a durable system you can build an asynchronous, client-side batching on top of Amazon SQS libraries that delays enqueue of messages to Amazon SQS and transmits a set of messages in a batch


# SNS

## ACG notes 
https://acloud.guru/course/aws-certified-solutions-architect-associate/learn/applications/sns/watch

#### Can push notifications as SMS to cell, email, SQS, http endpoints 
#### Topic is an access point that users can subscribe to. A topic can deliver to multple endpoint types. All SNS stored reduntantly accorss AZs.
#### instataneupus push based delivery
#### SNS is push based, SNS is pulled based using ec2s.

## SA Study Guide notes
#### SNS based on publish-subscribe (pub-sub) messaging paradigm
#### Apple, Android, Fire OS, and Windows devices, In China, to Android devices with Baidu Cloud Push. (SMS) messages to mobile device users in the United States or to email recipients worldwide

#### two types of clients: publishers and subscribers (sometimes known as producers and consumers)

![image1](https://learning.oreilly.com/library/view/aws-certified-solutions/9781119138556/images/ec08f004.jpg)

* relay events in workflow systems among distributed computer applications, 
* move data between data stores, or update records in business systems
* Event updates and notifications concerning validation, approval, inventory changes, and shipment status are immediately delivered to relevant system components and end users
* relay time-critical events to mobile applications and devices

* Application and System Alerts
* Push Email and Text Messaging
* Mobile Push Notifications, directly to mobile applications

### Fanout
#### parallel asynchronous processing by sending to multiple places. The illustration; order placed, one ec2 endpoint starts processing the order, 2nd endpoint ec2 send order data to DW app for analysis, or data can be replicted to dev env for further optimization.
![sns image](https://learning.oreilly.com/library/view/aws-certified-solutions/9781119138556/images/ec08f005.jpg)


