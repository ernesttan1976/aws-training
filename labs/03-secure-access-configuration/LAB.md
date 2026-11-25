# Lab 03 — Secure application access and configuration

**Status:** Prepared — gated; not the active lab  
**Current exercise when activated:** Level 1 — Guided implementation  
**Region:** `ap-southeast-1` (Singapore), unless a required feature is unavailable

## Learning objectives

- Configure programmatic AWS access with temporary credentials and the AWS SDK credential provider chain; do not distribute long-lived access keys to application code.
- Define a Lambda execution role and a Secrets Manager resource policy that authorize access to one named secret, and explain how identity-based and resource-based policies interact.
- Store sensitive configuration in AWS Secrets Manager, use its default AWS-managed KMS key for encryption at rest, and use TLS for AWS API calls.
- Retrieve a secret at runtime without exposing its value in logs, function output, source control, or deployment configuration.
- Validate access boundaries with AWS evidence, and remove lab resources safely.

## Scenario

A small service has a partner API token. Developers need programmatic access to deploy and inspect the service, while a Lambda workload needs the token at runtime. The token must not be baked into the function package, environment variables, or source repository. The team wants a short-lived, low-cost exercise that demonstrates role-based access and managed secret storage.

## Problem statement

Deploy a small Node.js Lambda function with AWS SAM. Store a **synthetic, non-functional test token** in AWS Secrets Manager. The function retrieves that secret on invocation and returns only a non-sensitive success indicator. A dedicated Lambda execution role can read the one configured secret and write its own logs; a resource policy on the secret names that role as an allowed principal.

This lab does not call a real partner service. Never use a real API key, password, customer record, or production credential as test data.

## Prerequisite knowledge

- Node.js, npm, AWS CLI profiles, basic IAM policy JSON, Lambda, and AWS SAM.
- A dedicated AWS sandbox account and an approved deployment identity. Prefer an IAM Identity Center or other temporary-credential profile; do not use root credentials or create IAM user access keys for this lab.
- Read the references on Lambda execution roles, IAM best practices, and Secrets Manager before deployment.
- Be able to inspect a SAM change set/template and recognize resources that are outside this lab's scope.

## Functional requirements

1. Configure the AWS CLI with a named sandbox profile and verify the active identity and region before deployment. The AWS SDK must use its standard credential provider chain; credentials must not be embedded in code, a committed `.env` file, or the Lambda package.
2. Create one Secrets Manager secret containing JSON with a fake `apiToken` value. Create it outside the SAM template so its value is not placed in CloudFormation parameters, outputs, or deployment logs. Do not pass a real credential.
3. Use the default Secrets Manager AWS-managed KMS key (`aws/secretsmanager`) for this short-lived lab. Do not specify a customer-managed KMS key. The secret value must be encrypted at rest by Secrets Manager and retrieved over the AWS service's TLS endpoint.
4. Deploy one Node.js Lambda function with a non-secret `SECRET_ARN` setting. The function retrieves the value at invocation using AWS SDK for JavaScript v3 and validates that the expected field exists.
5. Give the function a dedicated execution role trusted by `lambda.amazonaws.com`. Grant only the logging actions needed for its own log group and `secretsmanager:GetSecretValue` for the configured secret ARN. Do not grant `secretsmanager:*`, `kms:*`, or wildcard resource access for secret retrieval.
6. Attach a Secrets Manager resource policy that grants `secretsmanager:GetSecretValue` to the exact Lambda execution role for that secret. Enable the service's public-policy block when applying the resource policy. Review any other policies attached to the role; an additional broad identity policy can expand effective access.
7. The handler must not log or return the secret value, and must not include it in error messages. Log only a safe outcome such as `configurationLoaded: true` and a request identifier.
8. Test the live function using a small invocation count, inspect its logs and deployed policies, and then follow `CLEANUP.md`.

## Non-functional requirements

- No static AWS credentials or real third-party credentials in the repository, command history, build output, Lambda configuration, or evidence.
- The function can read the configured secret but has no `PutSecretValue`, delete, list, or administrative permissions.
- The secret resource policy identifies the Lambda role directly and is not public or cross-account.
- Keep function timeout and log retention short and appropriate to a diagnostic invocation. Do not log secret payloads, tokens, or personally identifiable information.
- The exercise remains observable: the Lambda invocation succeeds, its response is sanitized, and its log stream contains no token value.

## Constraints

- Use AWS SAM for the Lambda and IAM resources, AWS CLI for secret setup/inspection/invocation, and Node.js with AWS SDK for JavaScript v3 for application code.
- The deployment can use an existing approved SAM artifact bucket. Do not create a NAT gateway, VPC, API Gateway, database, rotation Lambda, or always-on compute for this lab.
- Use one secret and a handful of function invocations. Configure a short retention period for the Lambda log group.
- The expected cost is **approximately $0.01–$0.25 USD** for a one-hour, low-volume run plus the seven-day secret recovery window, assuming one secret, the AWS-managed Secrets Manager key, minimal logs, and no NAT gateway. This is a planning estimate, not a quote; confirm current Singapore prices and account billing behavior. Keep a **$1 USD maximum spend expectation** and stop for review if the deployment plan or expected charges exceed it. Secrets Manager has a per-secret charge; API requests and logs may also be billable.
- Do not create or deploy billable resources without explicit approval. No resources are created by preparing this assignment.

## Level progression for this competency

- **Level 1 — Guided implementation (prepared):** complete the milestones below and use the Level 1 hints. This is the exercise to begin only when the curriculum progression gate reaches Lab 03.
- **Level 2 — Independent implementation:** design a different small AWS workload that reads one sensitive configuration value from a managed secret store using temporary AWS credentials and least-privilege permissions. Implement from requirements alone, without reusing the Level 1 template. Explain the credential source, policy boundary, encryption choice, and how you verified no value leaked.
- **Level 3 — Troubleshooting:** diagnose a deliberately faulty version with access failures and/or secret exposure symptoms. The faulty implementation and evidence packet will be supplied when Level 3 is activated; do not treat this description as a repair recipe.
- **Level 4 — Optimization:** improve a working secret-consuming workload by measuring retrieval frequency/latency and balancing caching, rotation freshness, IAM scope, and cost. Provide before/after evidence and do not add paid resources without approval.

## Guided implementation milestones — Level 1

1. **Plan:** draw the human/developer identity → SAM deployment and the Lambda execution role → Secrets Manager secret. Mark which identity deploys resources and which principal retrieves the secret. Explain why those principals are different.
2. **Verify credentials:** select the named sandbox profile, confirm `aws sts get-caller-identity`, and confirm `ap-southeast-1`. Use a temporary-credential profile; do not put credentials in environment files or source.
3. **Prepare the synthetic secret:** create a protected local JSON file outside the repository containing only a fake token. Create the secret without passing its value as a literal command-line argument, then remove the temporary file. Do not commit or upload it.
4. **Implement:** create a SAM template with a dedicated execution role, narrowly scoped log permissions, a `GetSecretValue` policy scoped to the secret ARN, the Lambda function, and a non-public resource policy for the exact role. Keep only the secret ARN in function configuration.
5. **Review before deployment:** validate the template/policies, inspect the proposed resource list, confirm the selected AWS-managed key, region, log retention, and spend estimate. Request provisioning approval before deployment.
6. **Deploy and exercise:** deploy after approval. Invoke the function a few times. Confirm it succeeds, its result is sanitized, and no token appears in source, function configuration, command output, or logs.
7. **Prove boundaries and clean up:** inspect the role and secret resource policies; simulate the role's permission for the configured secret and an unrelated secret ARN. Record evidence in `TESTS.md`, then perform all cleanup and verification in `CLEANUP.md`.

## Acceptance criteria

1. **AC1 — Temporary programmatic access:** the CLI identity is verified with STS and the SDK uses the standard provider chain; no long-lived AWS access key or real partner credential is committed or packaged.
2. **AC2 — Managed secret storage:** the synthetic token is stored in Secrets Manager, encrypted at rest with the default AWS-managed key, and its value is absent from the SAM template, function environment variables, and deployment output.
3. **AC3 — Least-privilege workload access:** the Lambda execution role can call `GetSecretValue` for the one configured secret and cannot read an unrelated secret according to policy simulation. The secret resource policy grants access only to the intended Lambda role; the role has no secret write/admin or broad KMS permissions.
4. **AC4 — Runtime behavior and no disclosure:** the deployed Lambda successfully reads and validates the secret, returns only a safe status, and its CloudWatch log stream contains no token value.
5. **AC5 — Safe operation and cleanup:** deployed resources match the approved plan and region, the usage/cost expectation is within the stated ceiling, the stack resources and log group are removed, and the secret is scheduled for deletion and later verified absent after its recovery window.

## Official references

- [DVA-C03 exam guide — Domain 2: Security](https://docs.aws.amazon.com/aws-certification/latest/developer-associate-03/developer-associate-03-domain2.html)
- [AWS IAM security best practices](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html)
- [Lambda execution roles and least privilege](https://docs.aws.amazon.com/lambda/latest/dg/lambda-intro-execution-role.html)
- [AWS SAM Developer Guide](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/what-is-sam.html)
- [AWS SDK for JavaScript v3 credential provider chain](https://docs.aws.amazon.com/sdk-for-javascript/v3/developer-guide/setting-credentials-node.html)
- [Create a secret in AWS Secrets Manager](https://docs.aws.amazon.com/secretsmanager/latest/userguide/create_secret.html)
- [Secrets Manager best practices](https://docs.aws.amazon.com/secretsmanager/latest/userguide/best-practices.html)
- [Authentication and access control for Secrets Manager](https://docs.aws.amazon.com/secretsmanager/latest/userguide/auth-and-access.html)
- [Secret resource policies](https://docs.aws.amazon.com/secretsmanager/latest/userguide/auth-and-access_resource-policies.html)
- [Secret encryption and decryption in Secrets Manager](https://docs.aws.amazon.com/secretsmanager/latest/userguide/security-encryption.html)
- [IAM Access Analyzer policy validation](https://docs.aws.amazon.com/IAM/latest/UserGuide/access-analyzer-policy-validation.html)
- [AWS Lambda pricing](https://aws.amazon.com/lambda/pricing/) · [Secrets Manager pricing](https://aws.amazon.com/secrets-manager/pricing/) · [CloudWatch pricing](https://aws.amazon.com/cloudwatch/pricing/)

## Estimate

**Duration:** 2–3 hours, including policy review, deployment, evidence collection, and cleanup.  
**Cost:** approximately $0.01–$0.25 USD for the stated short, low-volume run and seven-day secret recovery window, subject to the assumptions above. Check current `ap-southeast-1` rates before deployment.

## DVA-C03 mapping

- **Domain 2 — Security** (26% of scored content in the current guide).
- **Task 2.1 — Implement authentication and authorization mechanisms for applications and AWS services:** Skills 2.1.2 (access control), 2.1.3 (programmatic access), 2.1.5 (authenticated AWS service calls), and 2.1.6 (permissions for IAM principals and resources). Temporary credential profiles also reinforce the task's federation concepts; this lab does not configure an identity provider.
- **Task 2.2 — Implement encryption and manage sensitive data by using AWS services:** Skills 2.2.1 (encryption at rest/in transit), 2.2.4 (encryption keys), and 2.2.7 (secret management services).
