# Lab 04 — Authenticated application API

**Status:** Prepared — gated; not the active lab

**Current exercise when activated:** Level 1 — Guided implementation

**Region:** `ap-southeast-1` (Singapore), unless a required feature is unavailable

## Learning objectives

- Implement a REST API with API Gateway and a Lambda proxy integration using AWS SAM.
- Authenticate application users with Amazon Cognito user pools and a Cognito authorizer.
- Enforce application authorization separately from authentication, using verified identity claims and a narrowly defined application role.
- Validate request bodies at the API boundary and handle authorization failures without leaking sensitive details.
- Gather live evidence for allowed and denied requests, inspect deployed configuration, estimate costs, and remove all lab resources.

## Scenario

A support platform is preparing a small request-intake API. Signed-in customers may submit a support request and inspect a minimal profile response. Only a member of the support-operations group may call a diagnostic endpoint. The prototype has no database yet: it returns a synthetic acknowledgement and must not retain or log the submitted message.

## Problem statement

Build a small API Gateway **REST API** with a Cognito user-pool authorizer, a Lambda handler, and a request model. Include a public health route, authenticated customer routes, and an operations-only route. The handler must make authorization decisions from claims verified by API Gateway; it must not trust a user ID, group, or role supplied in the request body or headers outside the authorizer context.

Use only fake test accounts and non-sensitive request content. This is a short-lived learning service, not a production support system.

## Prerequisite knowledge

- Node.js, npm, AWS CLI profiles, JSON/HTTP APIs, Lambda, and AWS SAM.
- Basic understanding of JWTs, identity claims, authentication versus authorization, and IAM execution roles.
- A dedicated AWS sandbox account and approved deployment identity. Prefer temporary AWS credentials; do not use root credentials or create IAM user access keys for the lab.
- Lab 01 concepts: Lambda integration, CloudWatch Logs, least privilege, and SAM deployment. Lab 03 concepts: AWS credentials and execution roles.
- Review the API Gateway, Cognito, and request validation references below before deployment.

## Functional requirements

1. Verify the named sandbox profile, active identity, and selected region before deployment. Application-user credentials and AWS deployment credentials are distinct; do not put either in source control.
2. Use AWS SAM to deploy one Cognito user pool, a public app client with no client secret, one Cognito user-pool authorizer, one API Gateway REST API, one Lambda function, and a short-retention CloudWatch log group. Keep deployment scope limited to these resources and SAM's required artifacts.
3. Provide these routes:
   - `GET /health` — public; returns a minimal health status.
   - `GET /profile` — requires a valid Cognito-issued token; returns the authenticated subject identifier from the verified authorizer claims, not from caller-supplied data.
   - `POST /requests` — requires a valid Cognito-issued token; validates a JSON body with a required category, title, and message against an API Gateway request model; returns a synthetic accepted response and does not persist the request.
   - `GET /admin/diagnostics` — requires a valid Cognito-issued token and an application authorization check for the `support-operations` group. A valid user outside that group receives `403`.
4. Create two synthetic test users in the pool: one ordinary member and one `support-operations` member. Use non-sensitive test passwords and a safe local authentication/test client that obtains an ID token for API calls. Do not place test passwords or bearer tokens in source, shell history, screenshots, or submitted evidence.
5. Configure API Gateway to validate the Cognito ID token before invoking protected routes. The Lambda must use only the verified authorizer claims for caller identity and group membership. Do not treat a decoded, unverified JWT or a client-supplied group as authorization evidence.
6. Return consistent, minimal responses: unauthenticated calls are rejected, authenticated non-operations users cannot access diagnostics, and malformed request bodies are rejected. Do not include stack traces, token contents, passwords, or submitted message text in responses or logs.
7. Give the Lambda a dedicated execution role with only the log permissions needed for its own log group. It needs no DynamoDB, S3, Secrets Manager, KMS, or administrative permissions.
8. Exercise each route and each authorization outcome with a small number of calls; collect evidence in `TESTS.md`; then follow `CLEANUP.md`.

## Non-functional requirements

- No real customer data, credentials, tokens, or secrets. Do not commit synthetic passwords, access tokens, token response files, or local credential material.
- Authentication is enforced at API Gateway for every protected route. Authorization is enforced for the operations route using claims from the verified token context.
- A user-supplied subject/group value cannot grant access or change which identity the API reports.
- Invalid input and access-denied responses are useful but do not disclose internal implementation details.
- Logs contain request IDs and safe outcome codes only; do not log authorization headers, JWTs, passwords, or request message content.
- Configure a short Lambda timeout and log retention. Do not enable API Gateway caching, custom domains, WAF, or other paid/production features for this exercise.

## Constraints

- Use AWS SAM for infrastructure and deployment, AWS CLI for inspection and user-pool setup, and Node.js with AWS SDK for JavaScript v3 where application/test code needs AWS APIs.
- Use API Gateway REST API (not HTTP API) so the lab can demonstrate API Gateway request models and request-body validation alongside a Cognito user-pool authorizer.
- Use only API Gateway, Cognito user pools, Lambda, CloudWatch Logs, and resources SAM requires. No database, NAT gateway, custom domain, hosted compute, or real external support integration.
- Use `ap-southeast-1` where available. No resource may be provisioned until the learner explicitly approves the reviewed deployment plan.
- Target a small number of requests and a one-hour live-resource window. Review the actual SAM change set/template and region before approval.
- **Estimated cost:** approximately **$0.01–$0.50 USD** for a one-hour, low-volume run, subject to current regional prices, account pricing/free-tier eligibility, log volume, Cognito feature configuration, and any shared deployment artifact charges. This is a planning estimate, not a quote. Set a **$2 USD maximum spend expectation** and stop for review if the plan or expected charges exceed it. Check current API Gateway, Cognito, Lambda, and CloudWatch pricing before deployment.
- Delete the user pool and its test users with the stack. Never attach a production user pool or a shared API to this exercise.

## Level progression for this competency

- **Level 1 — Guided implementation (prepared):** build the specified support-request API using the milestones below and Level 1 hints. Begin only when the curriculum gate reaches Lab 04.
- **Level 2 — Independent implementation:** design and build a different small authenticated API from the outcome and constraints alone. Choose an appropriate identity/authentication mechanism and authorization boundary; do not reuse the Level 1 template. Explain token validation, principal claims, request validation, and the proof for denied calls.
- **Level 3 — Troubleshooting:** diagnose a deliberately faulty API implementation and evidence packet supplied when Level 3 is activated. Possible symptoms will involve token validation, incorrect identity/role trust, request validation, or access-denied behavior; no repair recipe is provided in advance.
- **Level 4 — Optimization:** improve a working API by measuring an agreed baseline and evaluating latency, payload validation, log volume, error clarity, and least-privilege access. Demonstrate a measured improvement without adding paid/always-on infrastructure without explicit approval.

## Guided implementation milestones — Level 1

1. **Model trust boundaries:** draw the test user → Cognito → API Gateway authorizer → Lambda flow. Mark where token verification occurs, where the verified subject/group claims are read, and which role is an AWS IAM role versus an application user/group.
2. **Verify account and plan:** confirm the active sandbox identity and region. Validate the SAM template; inspect all resource types and the proposed change set. Confirm there is no database, NAT gateway, custom domain, or unrelated resource, and review expected costs.
3. **Design the contract:** write the route/method table, authentication requirements, authorization rule, request schema, and success/denial response shapes before implementation. Decide what safe diagnostic information can be returned.
4. **Implement infrastructure and handler:** define the pool and app client, authorizer, REST API methods and request model, Lambda integration, execution role, and log retention. The handler must consume the API Gateway authorizer context, not caller-provided identity fields.
5. **Review and request approval:** inspect the resulting resources, IAM policies, pool/app-client configuration, method authorization, and request model. Confirm test accounts are synthetic and no credential/token is embedded. Obtain explicit approval before deployment.
6. **Exercise deployed behavior:** create the two test users and group membership, then test public health; authenticated profile and request submission; missing/invalid token; malformed request; ordinary-member access to diagnostics; and operations-member access to diagnostics. Do not print or save tokens as evidence.
7. **Collect evidence and explain:** complete `TESTS.md`. Explain where authentication ends and authorization begins, why the subject/group claims are trustworthy, what happens for each denial case, and what the request validator does and does not guarantee.
8. **Clean up:** delete the SAM stack, verify the pool, users, API, function, role, and log group are absent, review delayed billing data, and record cleanup evidence in `CLEANUP.md`.

## Acceptance criteria

1. **AC1 — Authentication boundary:** the public health route works without a token; protected routes reject missing and invalid/expired tokens; a valid Cognito token reaches the protected handler.
2. **AC2 — Identity integrity:** `/profile` reports the subject from verified API Gateway authorizer claims. A caller-supplied subject or group does not alter the identity or access decision.
3. **AC3 — Application authorization:** an authenticated ordinary member receives `403` from `/admin/diagnostics`, while a member of the configured operations group succeeds. A missing/invalid token is rejected before it is treated as an authorization decision for a valid user.
4. **AC4 — Request validation and safe handling:** valid request JSON receives the synthetic acceptance response; missing/invalid fields are rejected by the configured API Gateway validator/model. Responses and logs contain no bearer token, password, or submitted message content. No request data is persisted.
5. **AC5 — Least privilege and cleanup:** the Lambda role contains only required log permissions and no unrelated service access; deployed resources match the approved plan; observed use stays within the stated spend expectation; and cleanup verification confirms lab resources are removed.

## Official references

- [DVA-C03 exam guide — Domain 1: Development with AWS Services](https://docs.aws.amazon.com/aws-certification/latest/developer-associate-03/developer-associate-03-domain1.html)
- [DVA-C03 exam guide — Domain 2: Security](https://docs.aws.amazon.com/aws-certification/latest/developer-associate-03/developer-associate-03-domain2.html)
- [DVA-C03 exam guide — Domain 3: Testing and Deployment](https://docs.aws.amazon.com/aws-certification/latest/developer-associate-03/developer-associate-03-domain3.html)
- [AWS SAM Developer Guide](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/what-is-sam.html)
- [API Gateway REST API Cognito user-pool authorizers](https://docs.aws.amazon.com/apigateway/latest/developerguide/apigateway-integrate-with-cognito.html)
- [API Gateway request validation](https://docs.aws.amazon.com/apigateway/latest/developerguide/api-gateway-method-request-validation.html)
- [API Gateway models and mapping templates](https://docs.aws.amazon.com/apigateway/latest/developerguide/models-mappings-models.html)
- [Amazon Cognito user-pool app clients](https://docs.aws.amazon.com/cognito/latest/developerguide/user-pool-settings-client-apps.html)
- [Using Amazon Cognito user-pool tokens with API Gateway](https://docs.aws.amazon.com/cognito/latest/developerguide/amazon-cognito-user-pools-using-tokens-with-identity-providers.html)
- [Amazon Cognito user-pool groups](https://docs.aws.amazon.com/cognito/latest/developerguide/cognito-user-pools-user-groups.html)
- [Lambda execution roles](https://docs.aws.amazon.com/lambda/latest/dg/lambda-intro-execution-role.html)
- [API Gateway pricing](https://aws.amazon.com/api-gateway/pricing/) · [Amazon Cognito pricing](https://aws.amazon.com/cognito/pricing/) · [AWS Lambda pricing](https://aws.amazon.com/lambda/pricing/) · [CloudWatch pricing](https://aws.amazon.com/cloudwatch/pricing/)

## Estimate

**Duration:** 3–4 hours, including API contract design, deployment review, runtime testing, evidence collection, and cleanup.

**Cost:** approximately $0.01–$0.50 USD for the stated short, low-volume run under the assumptions above. Confirm current `ap-southeast-1` pricing and account-specific pricing before deployment.

## DVA-C03 mapping

- **Domain 1 — Development with AWS Services** (30% of scored content in the current guide): Task 1.1, developing code for applications hosted on AWS, including AWS API integration and application behavior; Task 1.2, developing code for AWS Lambda and integrating Lambda with AWS services.
- **Domain 2 — Security** (26%): Task 2.1, implementing authentication and authorization mechanisms for applications and AWS services, including identity claims, access control, and IAM permissions for the Lambda workload.
- **Domain 3 — Testing and Deployment** (22%): Task 3.1, preparing application packages for deployment, reinforced by SAM template validation and deployment review. This is not a CI/CD lab.
