# 115 DAYS OF BIG DATA - Dec 8th target

# Domain 1: Collection

# Day1
# Kinesis Overview
>- managed alternative to apache kafka
>- good for app logs, metrics, iot, clickstreams, realtime BD
>- streaming processing framework integration(spark, NiFi, etc)
>- data auto replicated synnchronously to 3AZ
### Kinesis Services
>- Kinesis streams: low latemcy streaming ingest at scale
>- Kinesis analytics: real time analytic streams using SQL
>- kinesis firehose: load streams into S3, redshift, ElasticSearch, Splunk 

>![architecture](https://github.com/Kinnoshachi/notes/blob/master/resources/KinesisArchitecture.png)
>![acg2](https://github.com/Kinnoshachi/notes/blob/master/resources/kinesisUses.png)
>![acg](https://github.com/Kinnoshachi/notes/blob/master/resources/KinesisStreamBenefits.png)
>![loads](https://github.com/Kinnoshachi/notes/blob/master/resources/KinesisLoad.png)

## Kinesis Streams

>- streams are divided in ordered shards / partitions
>- default data retention 24h, up to 7d
>- ability to reprocess / replay 
>- multiple sources can consume same stream
>- realtime processing with large scale of throughput
>- data is immutable



>>### kinesis stream shards
>>- one stream many shards
>>- billing per shard
>>- batching available
>>- resharding: split or merge
>>- records are ordered per shard
>>![shard img]()
>>>### Kinesis Shard stream records
>>> 1. Data Blob:
>>>>- is the data added to stream via a producer
>>>>- max size of data payload after base64-decoding is 1mb
>>> 2. Partition/Record Key: 
>>>>- groups records in shards
>>>>- same key = same shard
>>>>- specified by the application putting data into stream  *Use highly distibuted key to avoid *hot partition*
>>> 3. Sequence number: 
>>>>- unique identifierfor for each record put into a shard, added by kinesis after ingestion
>>>>- acts like a unique key that identifies a data blob
>>>>- assigned when a producer calls `PutRecord(s)`
>>>>- cant use seq num to logically seperate data in terms of what shards they came from, must use partition keys for that 

### Kinesis Data Stream Limits
>#### Producer
>- 1MB/s or 1000 messages at write PER SHARD, otherwise `ProvisionedThroughputException`
>#### Consumer Classic
>- 2mb/s read PER SHARD accross all consumers
>- 5 API calls/s PER SHARD accross all consumers
>#### Consumer Enhanced Fan-Out
>- 2mb/s read PER SHARD per enhanced consumers
>- no API needed (push model)

# Day 2
# Kinesis Security
- control access via IAM policies
- Encrypt in flight via HTTP endpoints
- at rest via KMS
- client side must be done manually
- VPC endpoints available for Kinesis to access within VPC

# Day 3
# Kinesis Producers
##### how to produce data into kinesis
- Kinesis sdk
- kinesis producer library (KPL)
- kinesis Agent: linux program that runs on servers
- 3rd party libraries spark, log4j appenders, flume, kafka connect, nifi

#### kinesis producer sdk - `PutRecord(s)`
>- PutRecords uses batch and increses throughput => less http requests
>- `ProvisionedThroughputExceeded` if go over limits
>-  AWS Mobile SDK: Android, iOS etc
>-  `PutRecords` is recomended: low throughput, higher latency, simple API, AWS Lambda

>>#### Managed AWS sources for Kinesis Data Streams
>>- cloud watch logs
>>- AWS IoT
>>- Kinesis Daya Analytics > can prodoce back into streams
>>#### AWS Kinesis API - Exceptions
>>- when exceeding MB or TPS for any shard
>>- make sure you dont have hot shard
>>#### Exception Solution
>>- reties with backoff
>>- increase shards(scaling)
>>- ensure partition key is good/distributed

#### KPL (kinesis producer library)
>- easy to use and highly configurable C++ / Java library
>- used for building high performance long running producers
>- automated and configurable retry mechanism
>- synchronous or Asynchronous API(better performance for async)
>- submits metrics to cloudwatch for monitoring
>- 2 batching subsections (both turned on by default) increases throughput decrease cost
>>1. collect records and write to multiple shards in the same `PutRecords` API call
>>1. Aggregate - increased latency, can store multiple records in one record +1000 records per second. incresases payload size and throughout(maximize 1MB/s limit)
>- Compression must be implemented by user
>- KPL Records must be de-coded with KCL or special helper library
>- do not use if application/use case cannot have additional processing delay
>- `RecordMaxBufferedTime` max amount of time buffered for optimized payload size


>![i4](https://github.com/Kinnoshachi/notes/blob/master/resources/Screen%20Shot%202019-08-27%20at%201.12.09%20PM.png)

#### Kinesis Agent
>- monitor log files and send direct to streams
>- java based writen ontop of kpl
>- install in linux only

>#### features 
>- write from multiple directories and write to multiple streams
>- routing feature based on directory / log file
>- can pre-process data before sending to streams (single line, csv to json, log to json)
>- handles file rotation, checkpointing and retry upon failures
>- emits metrics to cloudwatch
>- great for aggrgating mass of logs


# Day 4
# Kinesis Consumers
##### how to consume data from kinesis
>- Kinesis sdk
>- kinesis producer library (KCL)
>- kinesis Agent: linux program that runs on servers
>- 3rd party libraries spark, log4j appenders, flume, kafka connect, nifi
>- kinesis firehose
>- AWS Lambda
>- enhanced fan-out

#### Kinesis Conumer SDK - `GetRecords`
>- Classic Kinesis - records are polled by cosumers from a shard
>- each shard has 2MB total agg throughput 
>- `GetRecords` returns up to 10mb of data(then throttle for 5 sec) or up to 1k records
>- max 5 `GetRecords` API calls  per shard p/sec = 200ms latency
>- if 5 consumers poll same shard, each can call api 1 tim per sec and recieve <400kb/s

#### Kinesis Client Library (KCL)
>- java first with other language support
>- Read records produced by KPL (de-aggregate)
>- share multiple shards with multiple consumers in one group "shard discovery"
>- checkpointing feature to resume progress if a consumer goes down, leveraging Dynamo DB (one row per shard)
>- provision througput for dynamo DB or on demand provisioning or Dynamo will throttle and slow down KCL

#### Kinesis Connector Library
>- older java library, leverages the KCL
>- Write data to:
>>* S3
>>* DynamoDB
>>* Redshift
>>* ElasticSearch
>> Kinesis firehouse can replace 

#### AWS LAmbda
>- can run light weight ETL to:
>>* S3
>>* DynamoDB
>>* Redshift
>>* ElasticSearch
>> anywhere
>- lambda consumer has a library to de-agg records from kpl
>- can be used to trigger notif/emails in realtime
>- has a configurable batch size to regulate throughput

# Day 5
# Kinesis Ehanced Fanout
#### works wit KCL 2.0 or Lambda
>- each consumer gets 2mb/s throughput per shard
>- 2o consumers get 40mb/s per shard
>- 2mb/s p/shard limit exceeded due to pushing data to consumers over HTTP/2, and has reuced latency (~70ms)
##### Enhanced Fanout vs Standard Consumers
>###### When use Standard
>- low number of consumers <3
>- can tolerate ~200ms latency
>- minimize cost
>###### When use Enhanced 
>- multiple consumers for same stream
>- low latency requirment 
>- higher cost
>- default limit 5 consumers using enhanced fan-out per data stream

# Day 6
# Kinesis Scaling
##### Kinesis Operations - Adding Shards
>- aka Shard splitting
>- can be used to increase stream capacity (1mb p/shard )
>- can be used to splie hot shard
>- old shard is closed and then deleted once data expires
#### Kinesis Operations - Merging Shards
>- decrease stream capacity and save costs
>- groups shards with low traffic
>- old shards closed and deleted when data expires
#### Kinesis Operations - Auto scaling
>- auto scaling is not a native feature, however can use lambda
>- API call to change number of shards is `UpdateShardCount`
![architecture]
#### Kinesis Operations - limitations
>- resharding cannot be done in parallel, plan capacity in advance
>- only one sharding operation at a time, takes a few sec
>- 1k shards, takes 30k seconds, 8.3 hrs to double to 2k shards
>- cannot:
>>* scale more than twice in 24h rolling period for each stream
>>* scale more than double shard count, or less than half
>>* more than 500 shards per stream


# Day 7
# Kinesis firehose
#### Overview
>- fully managed, autoscaling
>- near realtime (60sec minimum for non full batches)
>- loads data into:
>>1. Redshift
>>1. S3
>>1. ElasticSearch
>>1. Splunk
>- supports many data formats and can convert from JSON to parque/ORC (only for s3)
>- data transformations through lambda (ex. CSV=>JSON)
>- supports compression when target is S3(GZIP, ZIP, Snappy)
>- only GZIP is further loaded into Redshift
>- pay only for amount of data going through firehose
![kdf diagram](https://github.com/Kinnoshachi/notes/blob/master/resources/Screen%20Shot%202019-09-01%20at%202.44.35%20PM.png)
![kdf delivery](https://github.com/Kinnoshachi/notes/blob/master/resources/Screen%20Shot%202019-09-01%20at%202.50.30%20PM.png)

#### Data buffering
>- firehose recieves outputs from source and accumulates in the buffer
>- the buffer is flushed based on time and size rules
>- buffer size or buffer time: if reached gets flushed
>- can auto increase buffer size to increase throughput
>- min time set can be 1min, size few mb
#### Kinesis Data Streams vs Firehose
>##### Streams
>- need to write custom code (producer/ consumer)
>- real time (~200 ms latency for classic, ~70ms for enhanced fan out)
>- must manage scaling (shard splitting / merging)
>- storage for 1-7 D, replay capability, multi consumer
>- use with Lambda to insert data in realtime to other services ie elastic search
>##### Firehose
>- fully managed, sends to S3, Splunk, RedShift, ES
>- serverless data transforms with lambda
>- near realtime(lowest is 1min)
>- automated scaling
>- no data storage

# Day 7 - 8 - 9 - 10
# Archetecture build

![sarchitecture](https://github.com/Kinnoshachi/notes/blob/master/resources/Screen%20Shot%202019-09-01%20at%203.06.20%20PM.png)


![req1](https://github.com/Kinnoshachi/notes/blob/master/resources/Screen%20Shot%202019-09-01%20at%203.18.18%20PM.png)
![req2](https://github.com/Kinnoshachi/notes/blob/master/resources/Screen%20Shot%202019-09-01%20at%203.18.36%20PM.png)
![req3](https://github.com/Kinnoshachi/notes/blob/master/resources/Screen%20Shot%202019-09-01%20at%203.17.39%20PM.png)
![req4](https://github.com/Kinnoshachi/notes/blob/master/resources/Screen%20Shot%202019-09-01%20at%203.17.45%20PM.png)
![req5](https://github.com/Kinnoshachi/notes/blob/master/resources/Screen%20Shot%202019-09-01%20at%203.18.01%20PM.png)