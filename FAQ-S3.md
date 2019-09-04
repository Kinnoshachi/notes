S3 FAQ

S3 generally provides 11 9's of durability and 4 9's of availability.

File size minimum: 0 maximum: 5TB. A single PUT operation can upload 5GB though.

Multi-part upload allows you:

Upload files in parallel by uploading parts of the file up.
Resume parts that fail upload, so you don't lose all your progress.
Allows you to pause the upload.
Start using multipart upload at > 100MB.

Q: What are the S3 storage classes?

S3-Standard: Standard S3 storage class.
S3-RRS: Reduced redundancy. 99.99% instead of 11 9's.
S3-IA: Infrequent access. You still get instant access, but you get charged more.

It is read after write consistency for new PUTs.
It is eventually consistency for overwrite PUTs and DELETES.

default S3 bucket limit 100

You get charged for:

Storage
Requests
Data transfer
S3-Acceleration
Tag management


You do not get charged for:

All transfers into S3 are free.
Transfers between AZs via the copy request.
Transfers between EC2 to S3.
CloudFront to S3 transfer.

S3 versioning affect costs A lot. Each version stored is a separate full copy of the object. 5 version of a 1MB object is 5MB of storage costs. This can quickly add up. You can use lifecycle policies to mitigate this.

You can use these access controls to control permissions:
Bucket policies - To control permissions on a bucket wide level.
ACLs - Access control list to finely control permission on an object basis.
You can use this protocol to secure S3 data in transit:

HTTPS protocol
You can add extra security measure for data at rest with:

SSE-S3 - Uses an S3 secret key associated with your account and encrypt the data before uploading. The key is automatically rotated, handled and managed for you by AWS. AWS issues a new master key at least monthly.
SSE-KMS - Like SSE-S3 except you have more control over the secret key via KMS - Key Management Service.
SSE-C - C stands for Customer key. The key is usually an on-prem key.
Encrypt before uploading - Simply means you can write your software or logic and encrypt the data first.

S3 Endpoint? It's a "logical entity" or a route with the AWS network that allows you to directly upload to S3 without being routed out to the internet network. This enables you to upload to S3 within your VPC without having to expose any world access to the VPC.

Q: What is Amazon Macie?
This is pretty cool. It's an AI service that automatically finds sensitive data personally identifiable information or intellectual property and surfaces it for you. It's AI monitoring to protect you from authorized data access or leaks. You can connect it to Lambda to take actions.

Q: How can you protect your S3 data from being accidentally deleted?
Enable versioning - only the owner of the bucket can then delete all objects
Using MFA Delete capability
You can set up S3 replication between 2 accounts. The most accident proof way.

Q: What are the 2 ways you can get data into S3-IA?
A PUT operation with the x-amz-storage-class=STANDARD_IA header.
Use lifecycle policies to move objects from STANDARD to STANDARD_IA.

Q: Will using S3-IA be slower than S3?
Nope, latency and performance are not affected. You just get charged more if you access the data frequently.

Q: What's the cost difference between S3-IA vs S3?
One quick way to think about it is that with S3-IA you get charged 1/2 as much for storage, but you get charged about twice as much for access to the storage. So if you access the data infrequently, you'll save money. But if you access the data frequently, you're actually paying more 🤦

Q: What are the minimums you should know about with S3-IA?
The exact amounts are not as important as the concept itself.
Minimum number of days you'll be charged: 30 days.
Minimum object size you'll be charged: 128KB.

Q: What are the pros and cons with Glacier?
It's super cheap if do not need access to your data quickly. Access to the data takes a long time though. The standard access times are 3-5 hours.

Q: What are the 3 ways to access data via Glacier?
Expedited: 1-5 minutes - data transfer costs 3X and request costs 2X more than Standard
Standard: 3-5 hours - data transfer costs 4X and request costs 2X more than Bulk
Bulk retrievals: 5-12 hours

Q: What API do you use to access data Glacier?
The same S3 API. There is no Glacier API.

Q: What are Amazon S3 event notifications?
It's like a watch on S3 actions. You can watch PUTs, POSTs, COPYs, or DELETEs and hook them up to Amazon SNS, Amazon SQS, or directly to AWS Lambda.

Q: Can you use S3 to host a static website?
Yes. Only a static website though. For example, you cannot run PHP.

Q: What's useful about S3 tags? What's a downside?
You can tag S3 objects and do cool things like apply IAM policies, set up lifecycle policies, and customize storage metrics with them. A con is that you that the tags cost you money. Though it's really cheap: $0.01 for 10,000 tags/month currently.

Q: What is S3 Analytics – Storage Class Analysis?
This is a pretty cool feature. It automatically identifies objects or patterns on a daily basis in your bucket that makes sense to move to the S3-IA storage class. Then you can create a Lifecycle policy to save money. S3 Analytics cost a small amount of money but you make it back up easily with the right S3-IA Lifecycle policy.

Q: What is S3 Inventory?
calling the S3 API list and iterate through every object is slow and expensive. S3 Inventory solves this. It provides you with a CSV with a list of your objects based on an S3 bucket or a prefix. You can run it weekly or daily.

S3 uses CloudWatch Metrics

Q: What is S3 Lifecycle Management?
S3 Lifecycle Management are rules you can define that automatically move your objects to different storage classes. You can also use it to expire unfinished multipart uploads and save money that way too.

Q: What is Amazon S3 CRR?
It is S3 Cross-Region Replication. It allows you replicate an entire bucket to another region. Useful for DR backup.

What is Amazon S3 Transfer Acceleration?
It's a way to upload to S3 to using a special endpoint that gets your data onto S3 faster than using the internet network. You pay a little more of for it but it is faster.

Q: How large should the data be to consider S3 Acceleration vs uploading to CloudFront?

> 1GB = Consider using S3 acceleration.
< 1GB = You're probably fine with uploading to CloudFront.
Q: How large should the data be to consider Snowball vs S3 Acceleration?

> 1 week = If it takes more than a week to upload the data, it's faster to have AWS physically drive a truck to their data center for you.
Rule of thumb: 1 fully used 1Gbps line can move about 75TB in a week.
Q: Does S3 support IPv6?

Yes.
