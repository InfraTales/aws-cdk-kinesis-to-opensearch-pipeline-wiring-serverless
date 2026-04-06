# Cost Model

## Overview

This is a reference cost model. Actual costs vary by usage, region, and configuration.

## Key Cost Drivers

- Kinesis Data Streams with provisioned shards costs ~$10.95/shard/month at rest — at low throughput, SQS + Lambda would be 80-90% cheaper, but Kinesis preserves ordering and replay within 7 days which SQS standard cannot guarantee [inferred]
- OpenSearch Service (managed) adds a minimum ~$50-150/month for even a single-node dev domain; at small event volumes, DynamoDB + Athena alone would cover most query patterns without the operational overhead of index management [inferred]
- KMS encryption on every service (S3, DynamoDB, Kinesis, Firehose) adds per-API-call costs that are negligible at low scale but can add $20-60/month at millions of events/day; envelope encryption with a single CMK across services is the right call here [editorial]

## Estimated Monthly Cost

| Component | Dev (₹) | Staging (₹) | Production (₹) |
|-----------|---------|-------------|-----------------|
| Compute   | ₹2,000–5,000 | ₹8,000–15,000 | ₹25,000–60,000 |
| Database  | ₹1,500–3,000 | ₹5,000–12,000 | ₹15,000–40,000 |
| Networking| ₹500–1,000   | ₹2,000–5,000  | ₹5,000–15,000  |
| Monitoring| ₹200–500     | ₹1,000–2,000  | ₹3,000–8,000   |
| **Total** | **₹4,200–9,500** | **₹16,000–34,000** | **₹48,000–1,23,000** |

> Estimates based on ap-south-1 (Mumbai) pricing. Actual costs depend on traffic, data volume, and reserved capacity.

## Cost Optimization Strategies

- Use Savings Plans or Reserved Instances for predictable workloads
- Enable auto-scaling with conservative scale-in policies
- Use DynamoDB on-demand for dev, provisioned for production
- Leverage S3 Intelligent-Tiering for infrequently accessed data
- Review Cost Explorer weekly for anomalies
