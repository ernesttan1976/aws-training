# Lab 06 — Validation and evidence

Lab 06 is prepared but gated. Do not make an S3 API call, create a bucket, or upload an artifact until the lab is activated and the learner explicitly approves the reviewed account, region, bucket/prefix, artifact contents, and estimated cost. No application deployment is part of this lab. Keep evidence free of credentials and local secret/config files.

| Criterion | Validation procedure | Evidence to submit |
|---|---|---|
| AC1 — Application behavior | Run automated tests for health, valid request, malformed JSON, missing/invalid fields, and quantity boundaries. Run at least one valid and one invalid request through the local HTTP API. Check that invalid input is neither echoed nor logged. | Sanitized test output, local HTTP status/body examples using synthetic data, and confirmation that the request body was not logged. |
| AC2 — Configuration | Run tests or local invocations under both named configurations. Show stage output and demonstrate that the configured maximum changes acceptance behavior without editing source. Inspect all config files for secrets. | Sanitized configuration selection and paired result evidence; repository secret/config review. |
| AC3 — Automated/event tests | Run the unit suite without AWS credentials/network access. Run `sam validate`, `sam build`, a `sam local invoke` with a committed HTTP API v2 fixture, and `sam local start-api` smoke tests. | Command summaries, exit statuses, fixture file names, test report, and concise note on local emulation limits. Do not submit credential files or full environment dumps. |
| AC4 — Package quality | Inspect `.aws-sam/build` and the function archive. Confirm runtime, handler, dependencies, package size, and absence of `.aws`, `.env*`, local credential files, test reports, and unnecessary development content. | Package file listing or sanitized inspection report, size, SAM template validation/build result, and a checklist of excluded material. |
| AC5 — Approved packaged template | After explicit approval, run `sam package` to the approved bucket/prefix. Inspect the output template's artifact URI and check the uploaded object's key, size, and metadata. Confirm no `sam deploy`/CloudFormation stack was run. | Sanitized caller identity (account/ARN redacted where appropriate), region, bucket/prefix, package command summary, object metadata, output-template reference, and evidence no stack/application resources were created. Never include credentials. |
| AC6 — Scope, cost, cleanup | Count uploads/retries and inspect S3 objects before and after cleanup. Confirm lab-created bucket removal if applicable. Check billing/usage after its normal reporting delay. | Request/object ledger, cost assumptions, cleanup verification, no-deployment confirmation, and date/source of any billing check. |
| AC7 — Reasoning | Explain what unit tests, event fixtures, local invocation, local API smoke tests, and artifact inspection each establish; explain what only a deployed integration test could establish. | Learner's written explanation and answers to instructor follow-up questions. |

## Minimum local test matrix

1. `GET /health` returns status `200`, the health response shape, and the configured stage.
2. A valid `POST /validate` returns status `200` and the documented fields.
3. An `orderId` that is empty or exceeds 32 characters is rejected with the stable `400` error shape.
4. `quantity` is rejected when missing, not an integer, below 1, or above the selected `MAX_QUANTITY`; boundary values behave as documented.
5. Invalid JSON is rejected safely; the original body does not appear in stdout/stderr or response content.
6. Changing `APP_STAGE` and `MAX_QUANTITY` changes only the expected behavior and requires no source edit.
7. HTTP API payload-v2 event fixtures include required method/path/body context and invoke successfully through `sam local invoke`.
8. Tests run without network access or AWS credentials; no test relies on an actual deployed API or Lambda function.
9. `sam validate` and `sam build` pass. The built package contains required runtime files and excludes local credentials, secret files, and unrelated development artifacts.
10. The packaged template points at the approved S3 location; the uploaded object is within the size limit and is removed during cleanup.

## Evidence checklist

- [ ] Lab 06 was activated before any S3 upload or bucket creation.
- [ ] Learner explicitly approved the reviewed identity, region, bucket/prefix, artifact content, and cost expectation.
- [ ] All test data is synthetic; no secrets or credentials are in code, fixtures, config, logs, or evidence.
- [ ] Automated tests, `sam validate`, `sam build`, `sam local invoke`, and local HTTP smoke test are evidenced.
- [ ] API Gateway HTTP API payload format version 2.0 is demonstrated.
- [ ] Built/uploaded artifact contents were inspected before and after packaging.
- [ ] No application deployment or CloudFormation stack was created.
- [ ] Upload count, storage/request cost assumption, and cleanup are recorded.
- [ ] Learner distinguishes local emulation from cloud integration evidence.
