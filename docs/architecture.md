# Architecture Notes

## Overview

The stack wires a serverless data pipeline end-to-end in CDK TypeScript [from-code]: API Gateway receives events, Lambda validates and forwards them to Kinesis Data Streams [from-code], Firehose buffers and lands raw payloads into S3 [from-code], and Kinesis Analytics runs SQL transformations in flight before fanning out to OpenSearch for near-real-time search and DynamoDB for low-latency key lookups [from-code]. Glue crawlers catalogue the S3 raw zone so Athena can run ad-hoc queries without a warehouse [from-code]. SNS and SQS handle dead-letter routing and async fan-out [from-code], CloudWatch alarms cover iterator age and Lambda error rates [from-code], and KMS encrypts data at rest across S3, DynamoDB, and the Kinesis streams [from-code]. The non-obvious design choice is the dual-sink pattern — the same enriched stream feeds both OpenSearch and DynamoDB, trading write amplification for query flexibility without a secondary ETL job [editorial].

## Key Decisions

- Kinesis Data Streams with provisioned shards costs ~$10.95/shard/month at rest — at low throughput, SQS + Lambda would be 80-90% cheaper, but Kinesis preserves ordering and replay within 7 days which SQS standard cannot guarantee [inferred]
- OpenSearch Service (managed) adds a minimum ~$50-150/month for even a single-node dev domain; at small event volumes, DynamoDB + Athena alone would cover most query patterns without the operational overhead of index management [inferred]
- Firehose buffering (default 5 min / 128 MB) means data lands in S3 with a lag — acceptable for analytics but disqualifying for anything needing sub-minute alerting, which is why OpenSearch gets the hot path via Kinesis Analytics [inferred]
- Glue crawler re-runs are needed every time the S3 partition scheme changes — if Lambda output schema drifts without a corresponding crawler trigger, Athena queries silently return stale or incomplete results [inferred]
- KMS encryption on every service (S3, DynamoDB, Kinesis, Firehose) adds per-API-call costs that are negligible at low scale but can add $20-60/month at millions of events/day; envelope encryption with a single CMK across services is the right call here [editorial]