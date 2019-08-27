# 155 DAYS OF BIG DATA
# Domain 1: Collection

# Day1
## Kinesis Overview
- managed alternative to apache kafka
- good for app logs, metrics, iot, clickstreams, realtime BD
- streaming processing framework integration(spark, NiFi, etc)
- data auto replicated synnchronously to 3AZ
### Kinesis Services
- Kinesis streams: low latemcy streaming ingest at scale
- Kinesis analytics: real time analytic streams using SQL
- kinesis firehose: load streams into S3, redshift, ElasticSearch, Splunk 

### Kinesis Streams
-streams are divided in ordered shards / partitions
- default data retention 24h up to 7d
- ability to reprocess / replay 
- multiple sources can consume same stream
- realtime processing with large scale of throughput
- data is immutable

### kinesis stream shards
- one stream many shards
- billing per shard
- batching available
- number of shards can change over time (reshard / merge)
- records are ordered per shard
### Kinesis Shard stream records
- records made of data blob <1MB, serialized bytes
- record key: sent with a record, helps group records in shards. Use highly distibuted key to avoid *hot partition*
- Sequence number: unique identifierfor each record put in shard added by kinesis after ingestion
### Kinesis Data Stream Limits
#### Producer
- 1MB/s or 1000 messages at write PER SHARD, otherwise `ProvisionedThroughputException`
#### Consumer Classic
- 2mb/s read PER SHARD accross all consumers
- 5 API calls/s PER SHARD accross all consumers
#### Consumer Enhanced Fan-Out
- 2mb/s read PER SHARD per enhanced consumers
- no API needed (push model)

# Day 2
# Kinesis Security
- control access via IAM policies
- Encrypt in flight via HTTP endpoints
- at rest via KMS
- client side must be done manually
- VPC endpoints available for Kinesis to access within VPC
