# Hints — Lab 02: Data access patterns and lifecycle management

Use these in order. Stop as soon as one is enough to unblock you. Level 1 hints are for the first exercise after Lab 02 is activated. Hints for Levels 2–4 will be added when those exercises begin; they are intentionally not solutions to future levels.

## Level 1 — Guided implementation

1. **Start from calls, not tables:** list the exact reads and writes the app needs. For each read, identify its partition key and whether a range/order is required.
2. **Find the query boundary:** the ID lookup should target the primary key. A status browse needs a key path that begins with status; DynamoDB filters do not turn a scan into an efficient query.
3. **Keep binary data separate:** store attachment bytes in S3 and a stable object key plus report metadata in DynamoDB. A presigned URL is time-limited access, not a public bucket policy.
4. **Treat expiry in two layers:** the application checks `expiresAt` on reads; DynamoDB TTL later removes eligible items asynchronously. TTL timestamps are Unix epoch **seconds**.
5. **Scope lifecycle carefully:** an S3 lifecycle rule can target an attachment prefix. Its age-based expiration is asynchronous and should be verified as configuration, not tested as an immediate delete.
6. **Review index costs:** a secondary index stores another view of data and can add storage and write work. Include only the attributes and access paths the stated workload needs.
7. **Permissions:** separate deployment authority from app runtime permissions. Scope DynamoDB actions to the table/index and S3 object actions to the lab prefix; avoid wildcards where resource ARNs are available.
8. **Evidence:** combine app output with AWS-side inspection: table/index description, TTL status, bucket public-access/encryption/lifecycle configuration, and sanitized IAM policy.

## Questions to answer while working

- What query does each key and index support, and what would make DynamoDB perform a scan?
- What does a `FilterExpression` do to the work already performed by a query or scan?
- Why can an expired record still be returned by DynamoDB after its TTL time?
- What protects an attachment when a presigned URL is issued, and what limits that access?
- Which additional storage or write costs does the secondary index introduce?
- How will you prove the lifecycle policy is active without waiting for an object to expire?
