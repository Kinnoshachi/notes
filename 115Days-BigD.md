# 115 DAYS OF BIG DATA
# Domain 1: Collection

# Day1
## Kinesis Overview
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
>1. collect records and write to multiple shards in the same `PutRecords` API call
>1. Aggregate - increased latency, can store multiple records in one record +1000 records per second. incresases payload size and throughout(maximize 1MB/s limit)
>- Compression must be implemented by user
>- KPL Records must be de-coded with KCL or special helper library
>- do not use if application/use case cannot have additional processing delay
>- `RecordMaxBufferedTime` max amount of time buffered for optimized payload size


>![i4](https://github.com/Kinnoshachi/notes/blob/master/resources/Screen%20Shot%202019-08-27%20at%201.12.09%20PM.png)

## Kinesis Agent
>- monitor log files and send direct to streams
>- java based writen ontop of kpl
>- install in linux only

>#### features 
>- write from multiple directories and write to multiple streams
>- routing feature based on directory / log file
>- can pre-process data before sending to streams (single line, csv to json, log to json)
>- handles file rotation, checkpointing and retry upon failures
- emits metrics to cloudwatch
- great for aggrgating mass of logs


