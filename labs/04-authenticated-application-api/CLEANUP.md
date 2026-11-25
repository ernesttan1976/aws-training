# Cleanup — Lab 04: Authenticated application API

**Do this immediately after collecting runtime evidence.** Use the same approved AWS profile and region used for deployment. Confirm the stack name before deletion; do not delete a shared user pool, API, or artifact bucket.

1. Stop local test clients and any test loop. Remove local-only token/password files if any were created; do not use real credentials. Verify no token, test password, or test payload was committed.
2. Delete the SAM/CloudFormation stack and wait for completion:

   ```sh
   aws cloudformation delete-stack \
     --stack-name <stack-name> \
     --profile <sandbox-profile> \
     --region ap-southeast-1
   aws cloudformation wait stack-delete-complete \
     --stack-name <stack-name> \
     --profile <sandbox-profile> \
     --region ap-southeast-1
   ```

3. If a dedicated SAM artifact bucket was created, empty and delete it only after verifying it is exclusive to this lab. Never empty/delete a shared bucket.
4. Verify the stack and these lab resources are absent: Cognito user pool (including its test users and groups), API Gateway REST API/stage/authorizer, Lambda function and execution role, and CloudWatch log group. Confirm no API custom domain or other resource was created outside the approved plan.
5. Review the sandbox billing view after usage data becomes available. API Gateway, Cognito feature configuration, Lambda, CloudWatch Logs, and artifact storage can have charges; billing data may be delayed. An absent immediate charge is not proof that costs are final.
6. Record the cleanup time, deletion verification, billing follow-up, and any resource that remains below. Resolve stack deletion failures or unexpected resources before marking cleanup complete.

## Post-cleanup verification

- Stack name / region:
- Stack deletion completed at:
- Cognito user pool and test identities absent:
- API Gateway REST API / stage / authorizer absent:
- Lambda function and execution role absent:
- Lambda log group absent:
- Lab-only artifact bucket absent (or reason retained):
- Remaining resources / follow-up:
- Billing review date / observed usage:
