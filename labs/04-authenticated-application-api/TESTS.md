# Validation — Lab 04: Authenticated application API

Record the test case, HTTP status, sanitized response, AWS CLI inspection result, and timestamp for each check. Redact account-sensitive identifiers. Never submit a password, bearer token, Authorization header, token response, or support-message content. Runtime evidence is required; template inspection alone does not satisfy the acceptance criteria.

| Criterion | Validation | Evidence to capture |
|---|---|---|
| AC1 — Authentication boundary | Call `GET /health` without credentials. Call each protected route without a token and with a deliberately invalid or expired token. Call a protected route with a valid token issued by the lab user pool. | Sanitized route/status matrix showing public health success, protected-route rejection, and valid-token success. Do not include token strings or full request headers. |
| AC2 — Identity integrity | Call `/profile` as each synthetic user. Send a deliberately different subject/group value in an irrelevant request field or header and confirm the reported subject and authorization behavior do not change. | Sanitized response showing the authenticated subject and a code/configuration review showing identity is read from API Gateway's authorizer context. Redact subject values if desired; never include token contents. |
| AC3 — Application authorization | Call `/admin/diagnostics` with a valid ordinary-member token and a valid operations-member token. Also test no/invalid token separately. | Status results: ordinary member `403`; operations member success; missing/invalid token rejected. Include deployed Cognito group membership and handler authorization rule without credentials. |
| AC4 — Request validation and safe handling | Submit one valid JSON request and malformed cases (missing required property, wrong type/invalid field, malformed JSON). Inspect the API method's validator/model and Lambda logs. Search local test output/logs for the test message and any token/password without copying those values into evidence. | Status/response matrix; method/model inspection; safe log sample containing request ID/outcome only; confirmation no message, token, or password was found in responses/logs and no data store exists. |
| AC5 — Least privilege and cleanup | Inspect the Lambda role, stack resources, API methods/authorizer, user pool, and log retention. Compare with the approved plan. Run cleanup and verify absence of all lab resources. | Sanitized policy/resource evidence, stack listing before deletion, absent-resource checks after deletion, cleanup timestamp, and billing review/estimate when available. |

## Suggested manual checks

Use the approved profile and region explicitly; replace placeholders with the deployed outputs. These commands inspect metadata only and do not print an application token:

```sh
aws sts get-caller-identity --profile <sandbox-profile> --region ap-southeast-1
aws cloudformation describe-stacks --stack-name <stack-name> --profile <sandbox-profile> --region ap-southeast-1
aws cloudformation list-stack-resources --stack-name <stack-name> --profile <sandbox-profile> --region ap-southeast-1
aws apigateway get-rest-apis --profile <sandbox-profile> --region ap-southeast-1
aws apigateway get-authorizers --rest-api-id <rest-api-id> --profile <sandbox-profile> --region ap-southeast-1
aws apigateway get-method --rest-api-id <rest-api-id> --resource-id <resource-id> --http-method POST --profile <sandbox-profile> --region ap-southeast-1
aws cognito-idp list-user-pool-clients --user-pool-id <user-pool-id> --profile <sandbox-profile> --region ap-southeast-1
aws cognito-idp list-groups --user-pool-id <user-pool-id> --profile <sandbox-profile> --region ap-southeast-1
aws lambda get-function-configuration --function-name <function-name> --query '{FunctionName:FunctionName,Role:Role,Timeout:Timeout,Environment:Environment.Variables}' --profile <sandbox-profile> --region ap-southeast-1
aws iam list-attached-role-policies --role-name <execution-role-name> --profile <sandbox-profile>
```

Do not use a command that prints a token as evidence. If a local test tool must receive a token, keep it in memory and report only the HTTP outcome. API Gateway and Cognito configuration inspection supports the result but does not replace live allowed/denied requests.

## Learner evidence and assessment

- AC1:
- AC2:
- AC3:
- AC4:
- AC5:
- Learner's explanation of token validation, identity claims, application authorization, and request validation:
- Assessor notes / knowledge gaps:
