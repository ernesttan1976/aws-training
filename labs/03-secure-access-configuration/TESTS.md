# Validation — Lab 03: Secure application access and configuration

Record command output, AWS CLI responses, log evidence, and timestamps for each check. Redact account-sensitive identifiers. Never submit a credential or the synthetic token value. Replace placeholders with deployed names and the approved profile.

| Criterion | Validation | Evidence to capture |
|---|---|---|
| AC1 — Temporary programmatic access | Run `aws sts get-caller-identity --profile <sandbox-profile> --region ap-southeast-1`. Inspect the SDK initialization and repository for credential literals or committed secret files. | Sanitized STS identity output, credential-provider code/configuration, and secret scan/repository status. Do not paste credentials. |
| AC2 — Managed secret storage | Inspect the secret metadata in Secrets Manager and the Lambda configuration. Confirm the creation workflow omitted `--kms-key-id`, function configuration contains only `SECRET_ARN`, and the SAM template/output contains no secret value. | Sanitized `describe-secret` metadata and Lambda configuration showing only the ARN; explanation that omission selects the AWS-managed `aws/secretsmanager` key. Do not retrieve or paste the secret value for evidence. |
| AC3 — Least-privilege workload access | Inspect the function's execution role trust policy and attached/inline policies. Inspect the secret resource policy and verify `BlockPublicPolicy` was used. Simulate `secretsmanager:GetSecretValue` for the configured secret ARN and an unrelated secret ARN using the function role as policy source. | Sanitized role ARN and policy documents; simulator results `allowed` for the configured ARN and `implicitDeny` for the unrelated ARN; resource-policy output. Explain any other role/account policy that could broaden access. |
| AC4 — Runtime behavior and no disclosure | Invoke the deployed Lambda with a small test event; inspect response and its CloudWatch log group. Search output/logs for the known fake token in your own local environment without copying it into evidence. | Sanitized invocation response containing success status only, log event showing safe status/request ID, and confirmation the fake token was not found in logs or response. |
| AC5 — Safe operation and cleanup | Verify stack resources/region against the approved plan, assess observed usage against the estimate, and follow `CLEANUP.md`. Confirm the stack/log group are absent and the secret is scheduled for deletion; verify its final removal after the recovery window. | Sanitized stack/resource listing before deletion, absent-resource checks after deletion, the secret's `DeletedDate`, and a later check confirming it is gone; cleanup timestamp and billing review/estimate if available. |

## Suggested manual checks

Use the profile and region explicitly. These are examples; adapt resource names and avoid printing any secret values:

```sh
aws sts get-caller-identity --profile <sandbox-profile> --region ap-southeast-1
aws cloudformation describe-stacks --stack-name <stack-name> --profile <sandbox-profile> --region ap-southeast-1
aws lambda get-function-configuration --function-name <function-name> --query '{FunctionName:FunctionName,Role:Role,Environment:Environment.Variables}' --profile <sandbox-profile> --region ap-southeast-1
aws iam get-role --role-name <execution-role-name> --profile <sandbox-profile>
aws secretsmanager describe-secret --secret-id <secret-arn> --query '{Name:Name,ARN:ARN,DeletedDate:DeletedDate}' --profile <sandbox-profile> --region ap-southeast-1
aws secretsmanager get-resource-policy --secret-id <secret-arn> --profile <sandbox-profile> --region ap-southeast-1
aws iam simulate-principal-policy --policy-source-arn <execution-role-arn> --action-names secretsmanager:GetSecretValue --resource-arns <secret-arn> <unrelated-secret-arn> --profile <sandbox-profile> --region ap-southeast-1
aws lambda invoke --function-name <function-name> --cli-binary-format raw-in-base64-out --payload '{}' /tmp/lab03-response.json --profile <sandbox-profile> --region ap-southeast-1
```

The Lambda configuration command is safe only if the handler configuration contains the ARN and no value. Do not use `get-secret-value` to create evidence by printing the payload. Policy simulation checks the role's authorization; it does not replace the live Lambda invocation or prove that all external policy layers have been considered.

## Learner evidence and assessment

- AC1:
- AC2:
- AC3:
- AC4:
- AC5:
- Learner's explanation of credentials, effective permissions, encryption, and secret-handling choices:
- Assessor notes / gaps to reinforce:
