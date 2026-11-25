# Hints — Lab 03: Secure application access and configuration

Use these in order. Stop as soon as one is enough to unblock you. Hints for Levels 2–4 will be written when those exercises become active; do not use this file as a solution key for those levels.

## Level 1 — Guided implementation

1. **Broad:** Separate the identity that deploys the stack from the role the deployed function assumes. A function's role is its AWS identity; application code should not carry credentials.
2. **Credential source:** The AWS SDK v3 credential provider chain can obtain credentials from a configured AWS profile when you run locally and from the Lambda execution role when running in Lambda. Confirm which identity is active before making CLI calls.
3. **Secret boundary:** A secret ARN is configuration, not the secret value. Keep the ARN in the Lambda environment; keep the synthetic token in Secrets Manager. Create a local test file with restrictive permissions and never put the value directly on a shell command line.
4. **Role permissions:** Start with only `secretsmanager:GetSecretValue` on the one secret ARN and the minimum CloudWatch Logs actions for the function's own log group. The function does not need `ListSecrets`, `PutSecretValue`, delete actions, or direct KMS administration.
5. **Resource policy:** A resource policy names the role that can access the secret. Its allow is evaluated alongside identity policies and other account controls; review the role for any other broad grants. Enable Secrets Manager's public-policy block when putting the policy.
6. **Encryption choice:** If you omit a customer-managed key when creating the secret, Secrets Manager uses its AWS-managed `aws/secretsmanager` key. AWS recommends this for most cases and it has no key charge. A customer-managed key is a separate design with key-policy, `kms:Decrypt`, and cost/cleanup implications; it is outside this Level 1 scope.
7. **Runtime safety:** Parse the secret in memory and return a boolean/status only. Avoid logging the secret object, raw SDK response, thrown error details that contain payloads, or process environment values.
8. **Verification:** Inspect the deployed function's role ARN, the role policy, the secret resource policy, and the function environment. Use policy simulation on both the target ARN and an unrelated ARN; live invocation confirms the integration actually works.
9. **Deployment review:** Inspect the SAM template or change set before creating resources. A VPC or NAT gateway is not required for this lab and materially changes cost.

## Questions to answer while working

- Which principal supplies credentials during deployment, and which principal supplies them during Lambda execution?
- Why is `SECRET_ARN` acceptable in function configuration while the secret value is not?
- What does the secret resource policy contribute, and how can another identity policy affect effective access?
- Why can the function call Secrets Manager without directly receiving KMS key material?
- What could go wrong if a function caches the secret for a long time while an operator rotates it?
- What evidence demonstrates the token was retrieved without being disclosed?

Levels 2–4 are not active yet and have no implementation hints here.
