# Lab 07 — Release safely with CI/CD

**Status:** Prepared — gated; Lab 01 remains active pending learner evidence and assessment  
**Exercise when activated:** Level 1 — Guided implementation  
**Region:** `ap-southeast-1` (Singapore), unless a required service or integration is unavailable

## Learning objectives

- Build a source-controlled CI/CD pipeline with AWS CodePipeline and AWS CodeBuild.
- Automate application tests, SAM validation/build/package, and a development smoke test.
- Promote the same packaged artifact through development and production stages, with a manual production gate.
- Deploy Lambda versions through an alias using a canary strategy and CloudWatch alarms; demonstrate an automatic rollback.
- Explain the boundary between pipeline execution permissions, CloudFormation deployment permissions, and the Lambda execution role.
- Gather operational evidence and remove all lab-created resources safely.

## Scenario

The Order Check team has a locally tested serverless API and a deployment-ready SAM application. Manual releases have made it difficult to prove which source revision is running and to stop a bad release before customers see it. The team wants repeatable releases, a development check, an explicit production gate, and a bounded rollback path.

## Problem statement

Extend the Order Check application from Lab 06 into a small, source-controlled delivery workflow. Use a private GitHub repository connected to AWS CodePipeline through AWS CodeConnections. The pipeline must run tests and SAM checks, build and package the application once, deploy it to a development environment, run an HTTP smoke test, pause for human approval, and then deploy the exact same artifact to a production environment.

Use AWS SAM and CloudFormation for application infrastructure. Configure the Lambda application to publish versions behind an alias and use a CodeDeploy-managed canary deployment with CloudWatch alarms. In the development environment only, demonstrate a controlled failed release and verify that traffic returns to the prior version before correcting the change. Production must not receive the deliberately failing revision.

The initial pipeline/bootstrap stack and the two application stacks are live AWS resources. This is a gated preparation assignment: **do not provision them until Lab 07 is activated and the learner explicitly approves the reviewed account, region, resources, access, duration, and cost estimate.**

## Prerequisite knowledge

- Lab 06 assessed, including the Order Check handler, deterministic tests, SAM template, and local validation workflow. Lab 06 is currently prepared but gated; this dependency does not make Lab 07 active.
- Git, GitHub pull requests/branches, Node.js, AWS CLI profiles, CloudFormation, Lambda, IAM, and basic CloudWatch metrics/alarms.
- AWS SAM CLI and Docker installed locally; GitHub account access to create a private repository and authorize AWS CodeConnections.
- A dedicated AWS sandbox account and a temporary-credential deployment identity. Do not use root credentials or create long-lived IAM user access keys.
- Read the AWS CodePipeline, CodeBuild, CodeConnections, SAM deployment preference, and CloudFormation change-set references below before activation.

## Functional requirements

1. Keep the application source and pipeline definition under version control in a private GitHub repository. Use AWS CodeConnections for the source integration; do not place GitHub tokens or AWS credentials in the repository, build variables, or application configuration.
2. Define the pipeline and its roles/resources as infrastructure as code. Document the one-time source-connection authorization and the approved procedure for creating or updating the pipeline stack.
3. Configure an AWS CodePipeline V1 pipeline with these observable stages: source revision; automated test, SAM validation/build/package; development deployment; development HTTP smoke test; manual production approval; and production deployment. A failed earlier stage must prevent production promotion.
4. Run automated tests, `sam validate`, and `sam build` in CodeBuild. Package once and pass the resulting immutable packaged template/artifact through both environment deployments. Do not rebuild application source independently for production.
5. Deploy separate development and production application stacks. Use CloudFormation change sets (or an equivalent reviewed change mechanism) so the proposed resources and parameter differences can be inspected before execution. Keep environment-specific values non-secret.
6. Publish Lambda versions and route API Gateway integrations through a Lambda alias. Configure a canary traffic shift and at least one CloudWatch alarm that causes CodeDeploy to stop and roll back an unhealthy deployment. Use short CloudWatch log retention.
7. After a successful development deployment, run a bounded HTTP smoke test against the development API. Use synthetic request data only. A failed smoke test blocks the production approval stage.
8. The manual approval action must identify the source revision and intended production change. Approval of the pipeline action is separate from the learner's prior approval to create AWS resources.
9. In development only, introduce a controlled revision that fails the configured health/error alarm during its canary window. Capture CodeDeploy/pipeline evidence that rollback restored the previously working version. Correct the revision, rerun the pipeline, and promote only the corrected artifact.
10. Record the source revision, stage outcomes, change-set review, development smoke results, approval decision, canary/rollback result, production smoke result, resource inventory, and cleanup evidence in `TESTS.md`. Evidence must omit credentials and private tokens.

## Non-functional requirements

- Use temporary credentials for human/CLI access and service roles for CodePipeline and CodeBuild. Apply least privilege to the source, artifact bucket, build, CloudFormation, CodeDeploy, Lambda, logs, and alarm actions.
- The pipeline must fail closed: failed tests, build, development deployment, smoke test, or canary alarm must not advance a release to production.
- Build and test behavior must be deterministic. Use synthetic, non-sensitive inputs; never print the full environment, credentials, tokens, or unsanitized events.
- Enable encryption at rest for pipeline artifacts using the account's approved standard (SSE-S3 is sufficient for this lab unless account policy requires otherwise); keep artifacts private and expire or delete them during cleanup.
- Record which commit and artifact were promoted. Both environments must deploy the same packaged application artifact; only explicitly approved non-secret environment parameters may differ.
- Do not treat a green pipeline run as proof of zero cost or complete cloud correctness. Verify deployed endpoints, alarms, deployment history, and CloudFormation state directly.

## Constraints and cost controls

- Use AWS CodePipeline V1, CodeBuild, CodeConnections, CloudFormation, AWS SAM, Lambda, API Gateway HTTP API, CodeDeploy, CloudWatch Logs/Alarms, and a dedicated private S3 artifact bucket. No NAT gateway, database, custom domain, always-on compute, or unrelated service is in scope.
- Use `ap-southeast-1` when the required integrations are available. Confirm current service, CodeConnections, and pricing availability before deployment; do not switch region without recording the reason and getting approval.
- Use a small CodeBuild Linux environment, short function timeout, low test volume, and seven-day or shorter log retention. Keep the deployment exercise within one working session and delete the pipeline afterward so it cannot incur ongoing charges.
- Limit pipeline executions to the initial healthy release, one controlled development rollback, and one corrected promotion (plus only necessary retries). The canary rollback exercise must target development only.
- **Estimated duration:** 3–5 hours for implementation, pipeline runs, evidence, and cleanup, after Lab 06 is assessed.
- **Estimated cost:** approximately **$0.25–$5 USD** for a short exercise with a single V1 pipeline, a few small CodeBuild runs, small S3 artifacts, modest Lambda/API Gateway requests, short-lived logs, and a small number of alarms. This estimate is not a quote: CodePipeline pricing, CodeBuild duration/compute, CloudWatch alarms/logs, S3 requests/storage, regional prices, and account terms can change. Confirm current prices for `ap-southeast-1`; use **$5 USD as the maximum spend expectation** and pause if the reviewed plan could exceed it. Delete the active pipeline and other lab resources promptly; do not assume free tier.
- Preparing this assignment creates no AWS resources and authorizes no spend. Before any connection, stack, bucket, pipeline, build, or deployment is created/run, review the exact resource plan, account identity, region, likely cost and maximum, duration, and cleanup steps with the learner and obtain explicit approval. Never broaden permissions as a workaround for a denied action.

## Four exercise levels

### Level 1 — Guided implementation (first exercise when activated)

Implement this assignment using the milestones below and progressive hints in `HINTS.md`. Complete all evidence in `TESTS.md` and cleanup in `CLEANUP.md`. The learner performs and explains the implementation; the instructor assesses evidence and reasoning.

### Level 2 — Independent implementation

After Level 1 is assessed, receive an outcome-only brief for a different application and delivery workflow. Select the AWS CI/CD components, release gate, and rollback verification from stated requirements and constraints. The detailed workload and acceptance criteria will be provided when Level 2 is activated; no solution design is provided here.

### Level 3 — Troubleshooting

After Level 2 is assessed, diagnose a deliberately faulty pipeline and sanitized execution/deployment evidence. Identify the smallest set of causes behind the failed release or unsafe promotion, repair them, and prove the fix. The fault set and evidence packet will be provided when Level 3 begins; this assignment does not disclose the faults or their repairs.

### Level 4 — Optimization

After Level 3 is assessed, optimize a working release workflow against a supplied baseline. Measure improvements to feedback time, deployment risk, artifact handling, or cost without weakening required tests, approval gates, or rollback behavior. The workload and baseline will be provided when Level 4 begins.

## Guided implementation milestones — Level 1

1. **Map the release path:** draw source commit → tests/build/package → development deployment → smoke test → human approval → production canary. Mark the artifact boundary and every point where a failure must stop promotion.
2. **Design roles and ownership:** identify the human identity, CodePipeline role, CodeBuild role, CloudFormation/CodeDeploy service roles, and Lambda execution role. For each, list required actions and resource scopes before writing policies.
3. **Create the source and pipeline definitions:** put application and pipeline IaC in a private repository, connect it through CodeConnections, and define explicit stages/actions. Protect the release branch and avoid credential-bearing settings in source.
4. **Automate verification and packaging:** run the test suite, `sam validate`, and `sam build` in CodeBuild. Produce one packaged template and verify the same artifact can be referenced by both environment deployments.
5. **Configure deployments and rollback:** add development/production stack parameters, Lambda alias/version deployment, the canary policy, and a CloudWatch alarm. Review the generated change sets and failure behavior before deploying.
6. **Review the provisioning plan:** confirm caller identity, account, region, CodeConnections status, resource list, roles, artifact bucket configuration, alarms/log retention, maximum spend expectation, intended test run count, and cleanup plan. Obtain explicit approval before creating or running billable resources.
7. **Deploy the healthy release:** deploy development, run the smoke tests, inspect the manual approval details, approve the corrected release, deploy production, and verify the production endpoint and deployed revision.
8. **Prove a safe failure:** introduce the controlled failure in a development-only revision, observe the canary alarm/rollback, and confirm production remains unchanged. Correct the source and prove a subsequent healthy run can be promoted.
9. **Collect evidence and clean up:** complete `TESTS.md`; delete the application stacks, pipeline/bootstrap stack, artifact objects/bucket, and lab-only connection if owned and safe to remove. Verify resource absence and complete `CLEANUP.md`.

## Acceptance criteria

1. **AC1 — Source and repeatable pipeline:** a private, source-controlled pipeline starts from a recorded revision and runs CodeBuild tests, SAM validation/build/package, development deployment, development smoke test, manual approval, and production deployment in the intended order.
2. **AC2 — Artifact integrity:** a single packaged application artifact from the recorded revision is used for both environments; environment settings are explicit and non-secret. The artifact bucket is private and its contents are inspected.
3. **AC3 — Promotion safety:** an automated test or development smoke failure blocks the approval/deployment path; production is not updated by a failed development release. Evidence identifies the action and revision that stopped.
4. **AC4 — Canary rollback:** the healthy app deploys behind a Lambda alias with a canary strategy. A development-only failing version triggers the alarm and automatic rollback; the prior development version serves successfully afterward, and production remains on its previous version.
5. **AC5 — Successful corrected promotion:** after the fix, the pipeline completes development checks, a human approves the intended revision, production deployment completes, and synthetic HTTP smoke tests pass in production.
6. **AC6 — Security and cost:** no static AWS/GitHub credentials or secrets are committed or printed; IAM scopes and pipeline artifact access are reviewed; deployment activity stays within the approved region, duration, run count, and $5 spend expectation.
7. **AC7 — Cleanup:** lab-created application stacks, pipeline, CodeBuild project/roles, alarms/log groups, and artifact objects/bucket are removed or explicitly documented as pre-existing/shared and retained. Verification evidence and delayed billing review are recorded.
8. **AC8 — Reasoning:** the learner explains the stage boundaries, artifact promotion model, role separation, manual gate, alarm/rollback behavior, and what evidence is still required for a production-grade release.

## Official references

- [DVA-C03 exam guide — Domain 3: Testing and Deployment](https://docs.aws.amazon.com/aws-certification/latest/developer-associate-03/developer-associate-03-domain3.html)
- [Task 3.3 — Deploy code by using AWS CI/CD services](https://docs.aws.amazon.com/aws-certification/latest/developer-associate-03/developer-associate-03-domain3.html)
- [AWS CodePipeline User Guide](https://docs.aws.amazon.com/codepipeline/latest/userguide/welcome.html)
- [CodePipeline pricing](https://aws.amazon.com/codepipeline/pricing/)
- [CodePipeline V1 pipeline types](https://docs.aws.amazon.com/codepipeline/latest/userguide/pipeline-types-planning.html)
- [AWS CodeBuild User Guide](https://docs.aws.amazon.com/codebuild/latest/userguide/welcome.html)
- [CodeBuild pricing](https://aws.amazon.com/codebuild/pricing/)
- [AWS CodeConnections User Guide](https://docs.aws.amazon.com/dtconsole/latest/userguide/welcome-connections.html)
- [Connect to GitHub with CodeConnections](https://docs.aws.amazon.com/dtconsole/latest/userguide/connections-create-github.html)
- [CodePipeline manual approval action](https://docs.aws.amazon.com/codepipeline/latest/userguide/approvals.html)
- [AWS SAM Lambda deployment preferences](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/using-codedeploy.html)
- [Lambda traffic shifting with CodeDeploy](https://docs.aws.amazon.com/lambda/latest/dg/automating-deployment.html)
- [CodeDeploy automatic rollback](https://docs.aws.amazon.com/codedeploy/latest/userguide/deployment-steps.html)
- [CloudFormation change sets](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/using-cfn-updating-stacks-changesets.html)
- [AWS IAM security best practices](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html)
- [Amazon S3 pricing](https://aws.amazon.com/s3/pricing/) · [CloudWatch pricing](https://aws.amazon.com/cloudwatch/pricing/) · [Lambda pricing](https://aws.amazon.com/lambda/pricing/) · [API Gateway pricing](https://aws.amazon.com/api-gateway/pricing/)

## Estimate

**Duration:** 3–5 hours, after Lab 06 assessment.  
**Cost:** approximately $0.25–$5 USD for the short run described under Constraints. Verify actual Singapore prices and the account's CodePipeline/CodeBuild billing terms before approval.

## DVA-C03 mapping

- **Domain 3 — Testing and Deployment** (22% of scored content in the current exam guide; verify the current guide before exam preparation).
- **Task 3.3 — Deploy code by using AWS CI/CD services:** especially Skills 3.3.3 (IaC templates), 3.3.4 (deploy revisions with CodeDeploy), 3.3.5 (orchestrate with CodePipeline), 3.3.6 (canary strategy), 3.3.7 (automatic/manual rollback), and 3.3.8 (source control and releases).
- Reinforces **Task 3.1** (prepare deployable artifacts and environment-specific configuration) and **Task 3.2** (automate application tests) from Lab 06. Lab 07 extends those skills into controlled release and deployment.
