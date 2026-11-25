# Lab 07 — Validation and evidence

Lab 07 is prepared but gated. Do not create a CodeConnections connection, stack, bucket, pipeline, build, or application deployment until the lab is activated and the learner explicitly approves the reviewed identity, resources, region, expected run count, duration, maximum spend expectation, and cleanup plan. Keep evidence free of credentials, tokens, private source settings, and sensitive environment output.

| Criterion | Validation procedure | Evidence to submit |
|---|---|---|
| AC1 — Source and repeatable pipeline | Record the private repository, source commit, CodePipeline execution, action order, CodeBuild test/build/package results, deployments, smoke test, approval, and production outcome. Trigger a second run from a controlled commit to establish repeatability. | Sanitized pipeline execution and action details; commit SHA; test/build summaries; stage status and times. |
| AC2 — Artifact integrity | Inspect the package and S3 object metadata. Compare the artifact URI/key or digest used by development and production. Inspect both change sets and environment parameters. | Sanitized artifact listing, size/digest, template references, and change-set review notes. |
| AC3 — Promotion safety | Cause a test or development smoke check to fail safely. Confirm the pipeline stops before the manual production approval/deploy action and inspect production stack/version before and after. | Failed stage evidence, pipeline stage status, and unchanged production deployment/version evidence. |
| AC4 — Canary rollback | Deploy the controlled failing revision to development only, generate the planned synthetic test traffic, observe the configured alarm and CodeDeploy rollback, then call the development API. Confirm production did not change. | Alarm state/history, CodeDeploy deployment status and rollback event, successful post-rollback dev response, and production version evidence. |
| AC5 — Corrected promotion | Commit the fix, run a fresh pipeline execution, verify development smoke tests, inspect and approve the intended production change, then test the production endpoint. | Commit SHA, approval actor/time (or sanitized record), CodeDeploy/CloudFormation completion, and synthetic HTTP status/body examples. |
| AC6 — Security and cost | Inspect repository and build logs for secret exposure; review each service role and artifact bucket policy; compare actual resources/runs with the approved plan. Review cost/billing after reporting delay. | Sanitized IAM/action matrix, secret scan/repository check, resource ledger, approved estimate and actual-use notes, billing review source/date. Never include credential values. |
| AC7 — Cleanup | Follow `CLEANUP.md`; inspect CloudFormation stacks, CodePipeline, CodeBuild, CodeDeploy, alarms/log groups, S3 objects/versions, and any lab-owned connection. | Before/after resource inventory, deletion results, absence checks, and delayed billing follow-up. |
| AC8 — Reasoning | Explain artifact immutability, stage failure behavior, role separation, change-set review, approval semantics, canary metrics, rollback limits, and what the evidence does not establish. | Learner explanation and instructor follow-up responses. |

## Minimum validation matrix

1. The private source revision is identifiable and starts a pipeline run.
2. A failing automated test blocks deployment/promotion.
3. A healthy revision passes tests, `sam validate`, and `sam build`; the package is produced once and inspected.
4. The development deployment completes and its HTTP health and validation smoke checks pass using synthetic values.
5. A failed development smoke test or deployment blocks the production stage.
6. The production action remains pending until the named human approves the intended revision/change.
7. The same packaged artifact is referenced for the development and production stacks.
8. Lambda alias traffic shifting and the configured alarm are visible in the deployment configuration.
9. The deliberately failing development revision triggers CodeDeploy rollback; the prior development revision serves successfully afterward; production remains unchanged.
10. A corrected revision is subsequently approved and deployed to production; synthetic HTTP smoke checks pass.
11. Pipeline/build/deployment service roles and artifact storage are scoped and do not expose credentials or secrets.
12. Lab resources and artifacts are removed and absence is verified; billing is reviewed after the normal reporting delay.

## Evidence checklist

- [ ] Lab 07 was activated and the learner approved the reviewed provisioning plan before AWS resources were created or pipeline runs started.
- [ ] Account, region, maximum spend expectation, run count, duration, and cleanup plan were recorded before provisioning.
- [ ] Pipeline and deployment IaC are version controlled; no AWS or GitHub credential is stored in source or logged.
- [ ] Commit SHA and immutable package identity are recorded across both environments.
- [ ] Failed check was shown to block production promotion.
- [ ] Manual production approval and change-set review are evidenced.
- [ ] Development canary alarm and rollback are evidenced; production stayed unchanged during the failure drill.
- [ ] Corrected production deployment and synthetic smoke tests are evidenced.
- [ ] Service roles and artifact bucket access were reviewed.
- [ ] Pipeline, stacks, artifacts, log groups/alarms and lab-owned connection were removed or their retention is explicitly justified and approved.
- [ ] Learner distinguishes a successful pipeline execution from proof of production readiness.
