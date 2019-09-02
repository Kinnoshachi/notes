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

# SQS FAQs
### Service access and regions

    - Each Amazon SQS message queue is independent within each region
    - You can transfer data between Amazon SQS and Amazon EC2 or AWS Lambda free of charge within a single region.
    - When you transfer data between Amazon SQS and Amazon EC2 or AWS Lambda in different regions, you will be charged the normal data transfer rate.
### Queue sharing
    - to share a queue with someone else associate an access policy to the queue. SQS APIs policy statements:
        * AddPermission
        * RemovePermission
        * SetQueueAttributes
        * GetQueueAttributes
    - To share a message queue with an AWS user, provide the full URL from the message queue you want to share. The CreateQueue and ListQueues operations return this URL in their responses.
    - You can configure an access policy that allows anonymous users to access a message queue.
    - The permissions API provides an interface for sharing access to a message queue to developers. However, this API cannot allow conditional access or more advanced use cases.
    - SetQueueAttributes operation supports the full access policy language. For example, you can use the policy language to restrict access to a message queue by IP address and time of day. 
### Limits and restrictions
    - `MessageRetentionPeriod` for duration in seconds of retention
    - use the console or the `SetQueueAttributes` method to set the MaximumMessageSize attribute between 1,024 bytes (1 KB), and 262,144 bytes (256 KB)
    - To send messages larger than 256 KB, use the Amazon SQS Extended Client Library for Java. This library lets you send an Amazon SQS message that contains a reference to a message payload in Amazon S3 that can be as large as 2 GB. 
    - messages can contain up to 256 KB of text data, including XML, JSON and unformatted text. 
    - there is a 120,000 limit for the number of inflight messages for a standard queue and 20,000 for a FIFO queue. Messages are inflight after they have been received from the queue by a consuming component, but have not yet been deleted from the queue.
    - Queue names are limited to 80 characters, You can use alphanumeric characters, hyphens (-), and underscores (_). queue's name must be unique within an AWS account and region
### Compliance
    - SQS is PCI DSS Level 1 certified.
    -  If you have an executed Business Associate Agreement (BAA) with AWS, you can use Amazon SQS to build your HIPAA-compliant applications, store messages in transit, and transmit messages—including messages containing protected health information (PHI).
### Server-Side encryption (SSE)
    - SSE encrypts messages via KMS
    - To enable SSE for a new or existing queue using the Amazon SQS API, specify the customer master key (CMK) ID: the alias, alias ARN, key ID, or key ARN of the an AWS-managed CMK or a custom CMK by setting the KmsMasterKeyId attribute of the CreateQueue or SetQueueAttributes action.
    - Before you can use SSE, you must configure AWS KMS key policies to allow encryption of queues and encryption and decryption of messages.
    - SSE for a queue, you can use the AWS-managed customer master key (CMK) for Amazon SQS or a custom CMK.
    - To send messages to an encrypted queue, the producer must have the kms:GenerateDataKey and kms:Decrypt permissions for the CMK.
    - To receive messages from an encrypted queue, the consumer must have the kms:Decrypt permission for any CMK that is used to encrypt the messages in the specified queue. If the queue acts as a dead letter queue, the consumer must also have the kms:Decrypt permission for any CMK that is used to encrypt the messages in the source queue.
    - SSE encrypts the body of a message in an Amazon SQS queue. SSE doesn't encrypt the following components:
            - Queue metadata (queue name and attributes) 
            - Message metadata (message ID, timestamp, and attributes)
            - Per-queue metrics
    - SSE uses the AES-GCM 256 algorithm.
    - The AWS KMS per-account limit (100 TPS by default). calc example 300 seconds × 100 TPS / 1 IAM user = 30,000 queues
    - estimate KMS usage cost. API Requests per que = (billing period in sec) / (data key reuse period in 1 sec) * (2* (number of producing principals )+(number of consuming principls))
    - In general, producing principals incur double the cost of consuming principals. If the producer and consumer have different IAM users, the cost increases.

### Security and reliability
    - SQS has its own IAM like language
    - supports the HTTP over SSL (HTTPS) and Transport Layer Security (TLS) protocols. 
### FIFO queues
    - default, FIFO queues support up to 3,000 messages per second with batching, or up to 300 messages per second (300 send, receive, or delete operations per second) without batching.
    - it is possible to move to a fifo queue after creating a standard queue
    - Any duplicates introduced by the message producer are removed within a 5-minute deduplication interval.
### Features, functionality, and interfaces


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


# SWF
### Workflows
* distributed and asynchronous apps as workflows, can use sequential or parallel processing
#### workflow domains
* scope SWF resources
* must specify a domain for all compnents of a workflow (type, activity types) can have more than 1 workflow in a domain. 
#### workflow hisrtory
* record of every event change since workflow began. Event is discrete change in workflow execution state
#### Actors
1. workflow starter
* any app that can initiate workflow executions

2. decider
* activities inside a workflow can run sequentially, parallel, A/synchronously 
* decider, logic that coordinates tasks. schedulrs tasks and provides input data to workers, processes events that arrive while workflow is in progress and closes workflow when objective is completed.
3. activity workers
* a single cpu process that performes activity task. different types of activity workers process diffferent activity types. when worker is ready to prcoess new task it polls swf, after complete returns status to swf. 
#### Tasks
* provides activity workers and deciders with work assignments 
1. activity task
* chrge credit card, check inventory
1. AWS lambda task
* like activity task
1. decision task
* sends state to decider, contains workflow history. each decision task contains paginated view of entire workflow execution history.
#### task lists
* organizes tasks of a workflow, dynamic queue(automatically created). When a task is scheduled, you can specify what queueu(task list) to put it in. lists then route tasks to workers.

#### long polling
* decider and activity workers poll swf.
#### object identifiers
* identified by workflow type, qctivity type, decision and activity tasks, workflow execution
>- registered workflow type identified by domain name and version, call to `RegisterWorkflowType`
>- registered activity type identified by domain name and version, call to `RegisteredActivityType`
>- decision and activity task identified by unique task token, token generated by swf, returned with info about task in `PollForDecisionTask` or `PollForActivityTask`
>- single execution of workflow identified by domain, workflow id, and runid. domain and workflow id passed to `StartWorkflowExecution`, runid is returned by it
#### Workflow execution closure
>- workflow can be closed as complete, cancelled, failed, timed out. Can be continued as a new execution or terminated. Decider, admin, or SWF can ckose a workflow execution.
![execution](https://github.com/Kinnoshachi/notes/blob/master/resources/Screen%20Shot%202019-09-02%20at%2012.44.03%20PM.png)



