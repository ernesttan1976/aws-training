# Lab 06 — Package and test an AWS application

**Status:** Prepared — gated; Lab 01 remains active pending learner evidence and assessment  
**Exercise when activated:** Level 1 — Guided implementation  
**Region:** `ap-southeast-1` for S3 artifact storage; local development otherwise

## Learning objectives

- Build a Lambda application package with AWS SAM and inspect exactly which code and dependencies are included.
- Separate non-secret configuration from code and demonstrate distinct local settings for development and test.
- Create representative API Gateway HTTP API payload-v2 test events and validate both handler behavior and a locally emulated HTTP request.
- Automate unit and contract tests, including invalid-input and configuration cases; assess any AI-suggested tests rather than trusting them uncritically.
- Explain the gap between local validation and cloud integration testing, and create a packaged template without deploying application resources.

## Scenario

A small fulfilment team wants a serverless endpoint that validates synthetic order requests before a later system accepts them. Developers need a repeatable local test workflow and a deployment-ready artifact, but the team has not approved an AWS application deployment yet.

## Problem statement

Create **Order Check**, a Node.js AWS Lambda function exposed through an API Gateway HTTP API event in an AWS SAM template. It provides `GET /health` and `POST /validate`. Use synthetic request data only. Test locally, build the function package, and upload the packaged artifact to a dedicated approved S3 bucket (or an explicitly approved bucket and prefix). Generate a packaged SAM template that references the uploaded artifact. **Do not run `sam deploy`, create a CloudFormation stack, or create a live API or Lambda function.**

The request contract for `POST /validate` is `{ "orderId": "...", "quantity": 3 }`. A valid request has a non-empty `orderId` of at most 32 characters and an integer `quantity` from 1 through a configurable `MAX_QUANTITY`. The success response is `200` with `{ "valid": true, "orderId": "...", "quantity": 3, "stage": "..." }`. Invalid JSON or invalid fields return a consistent `400` response without echoing rejected input. `GET /health` returns `200` with `{ "status": "ok", "stage": "..." }`. Both routes use non-secret `APP_STAGE` configuration.

## Prerequisite knowledge

- Node.js/npm, JSON, HTTP request/response basics, unit testing, and AWS CLI profiles.
- Basic Lambda, API Gateway HTTP API, AWS SAM, and S3 concepts.
- AWS SAM CLI and Docker installed locally for `sam local` execution; Node.js 24.x installed for local tests/builds.
- Access to a dedicated AWS sandbox with temporary credentials and permission to write and remove objects in a dedicated private S3 bucket/prefix. Bucket creation is not required if an approved bucket already exists.
- Lab 01 concepts around Lambda request handling and safe error behavior are useful but not required to prepare this gated assignment.
- Lab 01 remains active until assessed. Do not make AWS calls for this lab unless it is activated. Before any S3 upload or bucket creation, obtain the learner's explicit approval after reviewing identity, region, bucket policy, artifact contents, and cost expectation.

## Functional requirements

1. Implement the two routes and the request/response contract above using Node.js 24.x, AWS SAM, and the API Gateway HTTP API payload format (version 2.0). Only synthetic test values may be used.
2. Define a SAM template for one Lambda function and its HTTP API events. Keep memory and timeout modest. Do not add databases, queues, custom domains, authorizers, NAT gateways, or unrelated AWS resources.
3. Provide two named non-secret configuration sets (for example `dev` and `test`) for `APP_STAGE` and `MAX_QUANTITY`. Show that changing configuration changes validation behavior without changing application source. Do not store credentials, passwords, API keys, tokens, or other secrets in template parameters, SAM config, environment files, source, or test fixtures.
4. Keep deterministic unit tests runnable without AWS credentials or network access. Include tests for both routes, valid and invalid payloads, malformed JSON, configured quantity limits, and missing/invalid configuration. Use stubs or mocks if any external dependency is introduced; no external service is required by this application.
5. Include one or more API Gateway HTTP API v2 event fixtures under version control. Fixtures must contain only synthetic data and should be close enough to the documented event contract to catch handler assumptions. Exercise at least one fixture with `sam local invoke`; use `sam local start-api` for a local HTTP smoke test.
6. Make test/build/package commands repeatable and documented. `sam validate` and `sam build` must complete successfully. Scope function source paths to the required application code, then inspect the build directory and package contents; exclude development-only files and all local credentials/configuration artifacts.
7. After learner approval, use `sam package` to upload the deployment artifact to the approved S3 bucket/prefix and write a packaged template to a local output path outside version control. Verify that template references the approved bucket/prefix and that the uploaded artifact is readable/has expected metadata. Do not deploy it.
8. Record the actual test commands/results and evidence in `TESTS.md`. Do not submit access keys, session tokens, complete credential files, or local environment files.

## Non-functional requirements

- Use least-privilege, temporary credentials for the packaging identity. The application itself requires no AWS SDK call or AWS credentials.
- Keep test behavior deterministic, fast, and repeatable. Treat local SAM execution as evidence of the local runtime contract, not proof of cloud IAM, deployment, or API Gateway service behavior.
- Do not log entire input events or echo invalid request bodies. Error responses should be stable and safe.
- Keep deployment artifacts small and inspectable. Do not upload `.git`, `.aws`, `.env*`, test output, `node_modules` when not needed, or any credential-bearing file.
- No live application infrastructure is required or permitted in Level 1; no CloudFormation stack or Lambda/API Gateway resources should exist from this lab.

## Constraints and cost controls

- Use Node.js 24.x, AWS SAM CLI, Docker for local SAM runtime emulation, and the built-in Node.js test runner (`node --test`) or an equivalent documented local test framework.
- Use API Gateway HTTP API payload format 2.0, not a REST API event. Do not substitute a manually constructed handler-only test for the SAM local event invocation.
- Use `ap-southeast-1` when creating an S3 bucket or using an approved artifact bucket. Check the bucket's actual region and access controls before packaging.
- Do not run `sam deploy`, `sam sync`, CloudFormation create/update, or any command that creates a running application resource.
- The S3 artifact must be no larger than 10 MiB. Keep the upload to a single package attempt plus only necessary retries. Remove the uploaded object and any lab-created bucket after evidence is collected.
- **Estimated duration:** 2–3 hours for implementation, local tests, package review, approved artifact upload, evidence, and cleanup.
- **Estimated cost:** expected **$0.01–$0.10 USD** for a single small artifact upload, short storage duration, and a few S3 requests; local compute and local SAM emulation do not create AWS service charges. Actual S3 rates, request type, region, free-tier/account terms, and minimum billing behavior can differ. Check current Singapore S3 pricing. Set a **$1 USD maximum spend expectation**; pause if the plan or actual usage may exceed it. Creating a bucket itself does not incur a recurring bucket fee, but its stored objects and requests can be billed.
- No upload or bucket creation may occur before Lab 06 is activated and the learner explicitly approves the reviewed plan. If no suitable approved bucket or permission exists, stop before the upload step; do not create IAM grants or broaden permissions as a workaround.

## Four exercise levels

### Level 1 — Guided implementation (first exercise when activated)

Implement Order Check as specified. Follow the milestones and progressive hints in `HINTS.md`, prove every criterion in `TESTS.md`, and complete cleanup. A live deployment is out of scope.

### Level 2 — Independent implementation

After Level 1 is assessed, prepare a different AWS application for deployment from an outcome-only brief. Choose the package/configuration boundaries, write a test plan, and provide deployment artifacts plus repeatable tests. Requirements and constraints will be issued when Level 2 begins; no solution design is provided here.

### Level 3 — Troubleshooting

After Level 2 is assessed, inspect a deliberately faulty SAM application and sanitized test/build/package evidence. Find the smallest set of causes behind failing tests or unsafe/missing configuration, repair them, and prove the fix. The starter, failure symptoms, and evidence packet will be issued when Level 3 begins; this document does not disclose the faults or repair.

### Level 4 — Optimization

After Level 3 is assessed, optimize a working application package and its validation workflow against a supplied baseline. Demonstrate a measured reduction in package size, test time, or configuration risk without reducing meaningful test coverage or adding unapproved infrastructure. The workload and measurements will be provided when Level 4 begins.

## Guided implementation milestones — Level 1

1. **Review boundaries:** inspect AWS SAM local testing, packaging, Lambda environment-variable guidance, and the DVA-C03 Domain 3 task statements. Draw the flow from source → tests → SAM build artifact → approved S3 artifact location → packaged template. Mark what is never uploaded.
2. **Define the contract:** write a route/method table, event version, valid and invalid request examples, status codes, and the two configuration sets. Use synthetic values only.
3. **Build the template and code:** define a minimal SAM function with `GET /health` and `POST /validate` HTTP API events. Use SAM parameters and Lambda environment variables for non-secret configuration. Keep request validation and response behavior deterministic.
4. **Automate tests:** write isolated unit tests for the contract and boundary cases, then add representative version-2.0 event JSON fixtures. Review any AI-suggested test cases and confirm each tests a real requirement; do not send secrets or private source to an unapproved model.
5. **Validate locally:** run the test suite, `sam validate`, `sam build`, `sam local invoke` with a fixture, and a local HTTP smoke test with `sam local start-api`. Inspect stdout/stderr and verify no input event is dumped.
6. **Review package contents:** inspect `.aws-sam/build` and the function archive. Confirm the runtime, handler, dependencies, environment configuration and event resources are correct; verify secrets, local credentials, test output and unnecessary files are absent.
7. **Review and approve the upload:** verify the AWS caller identity, account, S3 bucket name/region, bucket ownership/access controls, artifact size, estimated charge, prefix, and exact cleanup plan. Obtain explicit learner approval before running `sam package` or creating a bucket.
8. **Package without deploying:** upload to the approved S3 location, save a packaged template outside source control, inspect the S3 object and template reference, and stop. Do not run deploy/sync or create a stack.
9. **Evidence and cleanup:** complete `TESTS.md` and `CLEANUP.md`. Explain what local tests establish and what still requires cloud integration testing, and show that the uploaded object and any lab-only bucket were removed.

## Acceptance criteria

1. **AC1 — Application behavior:** `GET /health` and `POST /validate` follow the documented request/response contract for success, malformed input, and invalid values; rejected request content is not echoed or logged.
2. **AC2 — Configuration:** the two non-secret configurations can be selected independently of source changes, and test evidence shows that `APP_STAGE` and `MAX_QUANTITY` affect behavior as specified. No secret is stored in application configuration or the repository.
3. **AC3 — Automated and event-driven tests:** deterministic unit tests pass without AWS credentials/network access; API Gateway HTTP API v2 fixture(s) are exercised with SAM local invocation; the local HTTP smoke test passes. Test cases cover boundaries and failure paths.
4. **AC4 — SAM package quality:** `sam validate` and `sam build` pass; artifact inspection shows the correct handler/runtime/dependencies and no local credentials, private config, test output, or unnecessary development files.
5. **AC5 — Approved packaged template:** after explicit approval, `sam package` produces a packaged template referencing only the approved bucket/prefix and uploads an artifact no larger than 10 MiB. There is evidence that no application deployment/CloudFormation stack was created.
6. **AC6 — Scope, cost, and cleanup:** AWS activity stayed within the approved request and $1 expectation; there is no deployed Lambda/API/stack. The uploaded object and any lab-created bucket are removed, with evidence and delayed billing caveat recorded.
7. **AC7 — Reasoning:** the learner explains configuration versus secrets, what each test layer proves, SAM build versus package versus deploy, event contract fidelity, and the limitations of local emulation.

## Official references

- [DVA-C03 exam guide — Domain 3: Testing and Deployment](https://docs.aws.amazon.com/aws-certification/latest/developer-associate-03/developer-associate-03-domain3.html)
- [AWS SAM Developer Guide](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/what-is-sam.html)
- [AWS SAM CLI — `sam validate`](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/sam-cli-command-reference-sam-validate.html)
- [AWS SAM CLI — `sam build`](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/sam-cli-command-reference-sam-build.html)
- [AWS SAM CLI — `sam local invoke`](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/using-sam-cli-local-invoke.html)
- [AWS SAM CLI — `sam local generate-event`](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/using-sam-cli-local-generate-event.html)
- [AWS SAM CLI — `sam local start-api`](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/using-sam-cli-local-start-api.html)
- [AWS SAM CLI — `sam package`](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/sam-cli-command-reference-sam-package.html)
- [Lambda environment variables](https://docs.aws.amazon.com/lambda/latest/dg/configuration-envvars.html)
- [API Gateway HTTP API Lambda integrations and payload formats](https://docs.aws.amazon.com/apigateway/latest/developerguide/http-api-develop-integrations-lambda.html)
- [Node.js Lambda deployment package](https://docs.aws.amazon.com/lambda/latest/dg/nodejs-package.html)
- [Amazon S3 pricing](https://aws.amazon.com/s3/pricing/)

## Estimate

**Duration:** 2–3 hours, including automated tests, local SAM execution, package inspection, approved upload, evidence, and cleanup.  
**Cost:** expected $0.01–$0.10 USD for one small artifact upload and short-lived S3 storage under the assumptions above. This is an estimate, not a quote. Confirm current regional pricing and the actual bucket/request plan before approval.

## DVA-C03 mapping

- **Domain 3 — Testing and Deployment** (22% in the current exam guide; verify the guide before exam preparation).
- **Task 3.1 — Prepare application artifacts to be deployed to AWS:** particularly Skill 3.1.1 (manage code-module dependencies and configuration within packages) and Skill 3.1.4 (manage environment-specific configuration using AWS services such as Lambda/SAM).
- **Task 3.2 — Implement and automate application testing in AWS environments:** Skills 3.2.1 (develop and validate tests using AWS SAM/IaC tools), 3.2.3 (create Lambda/API Gateway application test events), and 3.2.5 as an optional extension (review AI-generated test ideas and automate repetitive test execution).
- This is a package-and-test lab, not a deployment or CI/CD pipeline lab; Domain 3 Task 3.3 is reserved for Lab 07.
