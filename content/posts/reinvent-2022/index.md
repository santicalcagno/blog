+++
title = "Notes on re:Invent 2022 talks I attended"
date = 2022-12-16
slug = "reinvent-2022"
+++

Here's a rough dump of all the notes I took while attending AWS' re:Invent this year. I'm focusing on Chalk Talks, which are not recorded and usually have a smaller live audience, but I also included links to the Breakout Sessions if you're interested in watching them. I hope this is useful to someone!

## Serverless stream processing with AWS Lambda, Amazon Kinesis & Kafka (SVS402)

### Kinesis

The talk began with a discussion on a car data collection architecture using Kinesis Data Streams/Lambda/Firehose/DynamoDB/S3

Custom metric: Records/Batch

- Tune parameter: batch size

At the shard level, λ polls shard 1/sec and invokes synchronously with batch of messages. If it succeeds it goes on to the next batch, if errors, depends on how the event source map is configured:

- Default: invoke with same set of records until success or records age out of stream
- Report batch item failures: λ returns first failed item, advances pointer to that record and proceeds with other error handling
- Bisect: Split in half, try to process oldest half, if errors do the same until erroring record is found and process according to max retries and age
  - Retries that are part of the bisecting process do not count towards max
- Max retries/age: send to failure destination or discard

(Same for DDB streams 👆)

Some other comments made:

- There's a 1-1 between Kinesis shards and Lambda concurrency
- You (probably) don’t want the default event source mapping error handling
- Lambda metrics return success even though retry logic is invoked
- Async destination error in Lambda config is different from source mapping config (same as retries)

New stuff: [event filtering](https://aws.amazon.com/blogs/compute/filtering-event-sources-for-aws-lambda-functions/)

- Batch size and window are applied before filtering
- Reduces costs (filter is applied before λ)

Keep Lambda concurrency limits in mind when using Kinesis on demand

Some words on [parallelization factor](https://aws.amazon.com/blogs/compute/new-aws-lambda-scaling-controls-for-kinesis-and-dynamodb-event-sources/) (note: keeps order, "shard of shards", won't work if all/most shards have same pkey)
￼

A performance step-by-step:

1. Ensure working system
1. Measure performance: time to process 1 message, average batch
1. Tune producer
1. Tune λ: code, powertune with batch
1. Tune event source mapping: batch size/window, [enhanced fan-out](https://aws.amazon.com/blogs/aws/kds-enhanced-fanout/), parallelization factor, error handling

Max performance: ignore error config in ESM and retries and log all to CloudWatch

Tune λ memory size if slow

- [Lambda power tuning tool](https://docs.aws.amazon.com/lambda/latest/operatorguide/profile-functions.html)

Kinesis producer library not recommended for Lambda in speakers opinion (at least for Python)

### MSK

Scenario: fraud detection (transactions stream)

- Using Step Functions to process each batch: map state of store in DDB, detect using [Fraud Detector](https://aws.amazon.com/fraud-detector/), update DDB with prediction, if suspicious publish to SNS
- [Express workflow](https://docs.aws.amazon.com/step-functions/latest/dg/concepts-standard-vs-express.html) in Step Functions to resolve quickly

- Error handling is not as complete as Kinesis (yet)
- Scaling mechanism is different: no parallelization
- Has event filtering as well

AWS configures a "poller" which is the thing that pulls and sends messages to the function. Starts with 1 poller and 1 λ and then scales pollers vertically and horizontally according to the need (up to same number of partitions)

Metrics to monitor

- Offset lag
- λ errors
- λ invocations
- λ duration
- λ concurrency

Multi region: Kinesis (producer writes to 2 Kinesis), Kafka (has replication)

## Migrate from a relational database to Amazon DynamoDB (DAT306)

Relational DB challenges

- Hard to scale
- Accepts unbounded work
- Monolith: shared everything

Related to above

- Large scale joins are taxing
- Unpredictable performance: buffer cache vs data
- App code in DB means that code takes space in DB (stuff must be evaluated each time a query runs and so on)

Should you migrate?

- Yes: K/V lookups, 1:N queries, serverless apps, "internet-scale" apps, JSON documents
- No: ERP apps, data warehouses, data marts

Steps

- Document needed access patterns: name, input args (might be PKs), single or set, returned columns, frequency per hour (priority for each access)
- Relational to single table: data shapes (note: speaker is a defendant of single table philosophy)
  - Understanding table data: item size, item count, velocity (cool/hot), type (dimension/fact)
  - Deciding how to combine many entities into single table
    - Denormalized join: join tables w/inner join
      - Data shape: wide with dupe column values across tables
      - DDB usage pattern: Get-Item (single read, low latency)
    - Stacked with union: Union all queries filling with null for data not in each table
      - Data shape: different entities on same table, storage efficient
      - DDB usage pattern: Query (item collection returned, may need multiple reads)
- PK and SK
  - Good idea to add SK even though not needed yet
  - Possible PK choices: high cardinality, table PK, parent id, collection id, uuid
  - Possible SK choices: same as PK, static 0, line item id, iso or epoch date, date.now(), uuid
  - Decorate key and item for readability: add prefixes (i.e. pr- for product, ver- for version)
  - Duplicate date for queries, future use (product id as pk, product id as product)
  - Date formats iso8601, unix (for TTL - think about it on day one, even if you don't need it now)
  - Concatenated columns for index SK (hierarchy, similar to iso date)
- Sparse columns for sparse GSIs (SELECT CASE WHEN product = 'car' AND price > 70000 THEN price ELSE NULL END AS luxury)
- Align PK attributes for item collections: benefit of single table design (single query)
- Align other attributes for GSIs

Migration strategies

- Use DMS, Glue or ETL tools
- Write and run scripts to gain exp. with bulk loads
- Archive legacy data and migrate only dimension data, starting fresh
- Write to both DBs for some time for verification
- DMS Limitations: does not combine tables, can be used after preparing data with a VIEW, each thread writes at 200 WCU (12MB/min)
- Avoiding throttles
  - DDB sheds traffic for two reasons: throughput beyond provisioned limits, hot key limits (>1000 WCU/s for any one PK)
  - Avoid writing to any one PK faster than 1000/sec, regardless of table capacity
  - Use round robin shuffle to fan out heavy writes to each PK
- Staging data to S3 -> DDB
  - Custom code (Lambda/Container/EC2)
  - New: [DDB import from S3](https://aws.amazon.com/blogs/database/amazon-dynamodb-can-now-import-amazon-s3-data-into-a-new-table/)
    - Free GSI WCUs when loading, no WCU usage (!)

## Architecting for resiliency and high availability (ENU303)

IIRC most of this talk's content is already covered by preparing for the Solutions Architect exam (setting up things across AZs and regions, that kind of thing). My takeaway from it was this set of tools:

- [Elastic DR](https://aws.amazon.com/disaster-recovery/)
- [Service Catalog AppRegistry](https://docs.aws.amazon.com/servicecatalog/latest/arguide/intro-app-registry.html)
- [Resilience Hub](https://aws.amazon.com/resilience-hub/)
- [DevOps Guru](https://aws.amazon.com/devops-guru/)

## Running containerized workloads on AWS Graviton–based instances (CMP408)

Question on how to convince devs to use Graviton–based

- Top down: costs
- Bottom up: modernization (app tech stack i.e. upgrade Java SDK, libs)

Even though ie 10% slower, if cost is 20% lower and SLO is OK that’s still a win

Discussion on how to build multi-arch containers with pipelines that look like [this](https://aws.amazon.com/blogs/devops/creating-multi-architecture-docker-images-to-support-graviton2-using-aws-codebuild-and-aws-codepipeline/)

- Native compile: install docker on Graviton/Apple silicon, build and push as usual, buildx/CodeBuild/CI runner can control Graviton instance remotely
- Cross compilation: compile arm64 target on x86 host, C/C++ (x-comp available) vs Go/Rust (built-in), copy foreign binaries, no run during build
- Docker buildx: Use own existing x86 HW, emulates arm64 in SW, best for containers requiring minimal processing to build

Having multi arch containers is good when combined with spot -> can use any arch instance for lower price

Can define ECS Fargate task as Linux ARM64 to use Graviton

Is there a tool to check for Graviton readiness? No, nothing automatic

When not to use Graviton? (I wonder if most of this holds with the new instance types, WS still does of course)

- CPU bound (frequency) monoliths that only scale vertically
- GPU intensive (ML)
- Windows Server

## Build resilient applications using Amazon RDS and Aurora PostgreSQL (DAT316)

Some more tools/configs!

- [Cluster cache management](https://aws.amazon.com/blogs/database/introduction-to-aurora-postgresql-cluster-cache-management/)
- [pg_prewarm](https://www.postgresql.org/docs/current/pgprewarm.html)
- [AWS Fault Injection Simulator](https://aws.amazon.com/fis/) (!)

Timeouts

- Timeout layers
- Watch out for connection storms -> set lowest timeout time at DB level and increase up the stack

Global resiliency

- More manual (intended), async replication, data loss
- Managed RTO (Aurora)
- Aurora [write forwarding](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/aurora-global-database-write-forwarding.html), storage replication

JDBC driver is topologically aware -> handles DB connection failover scenarios better

- No such things in other langs yet

Postgres connections: around hundreds connections, not thousands

Future: [RDS blue-green deployments](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/blue-green-deployments-overview.html)

## Online talks

Lots of cool things on these ones.

### [Observability the open-source way (COP301)](https://www.youtube.com/watch?v=2IJPpdp9xU0)

### [Deploy modern and effective data models with Amazon DynamoDB (DAT320)](https://www.youtube.com/watch?v=SC-YAPgJpms)

### [Best practices for advanced serverless developers (SVS401)](https://www.youtube.com/watch?v=PiQ_eZFO2GU)

### [From RDBMS to NoSQL (sponsored by MongoDB) (PRT314)](https://www.youtube.com/watch?v=eEENrNKxCdw)

### [Building observable applications with OpenTelemetry (BOA310)](https://www.youtube.com/watch?v=efk8XFJrW2c)

- [CloudWatch RUM](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/CloudWatch-RUM.html)
- [CloudWatch Evidently](https://aws.amazon.com/blogs/aws/cloudwatch-evidently/)

### [Building containers for AWS (CON325)](https://www.youtube.com/watch?v=S7JwFFZ-7_Q)

Most I knew, but I'm taking this with me: [ECR pull-through cache](https://docs.aws.amazon.com/AmazonECR/latest/userguide/pull-through-cache.html)

### [Introducing Trusted Language Extensions for PostgreSQL (DAT221)](https://www.youtube.com/watch?v=gejPKbPQh74)

### [Build scalable multi-tenant databases with Amazon Aurora (DAT318)](https://www.youtube.com/watch?v=tWRhDxcMLYo)

I asked the speaker why they didn't opt for DMS for the migration process, the answer was that their DBAs had (presumably much more) experience with standard MySQL tooling.
