# Lab 05 — Cleanup

This lab uses local code and a small number of Amazon Comprehend API calls. It should not create persistent AWS resources. Do not delete or alter a shared role, profile, or policy used by other work.

## Local cleanup

1. Stop the CLI/test process and remove temporary fixture files or generated outputs that contain the synthetic source text.
2. Remove any local debug captures, request dumps, or scratch files. Keep only sanitized test reports and evidence.
3. Review the repository diff and confirm no credential files, local environment files, raw fixtures, or logs were added. Ensure `.gitignore` covers local-only secrets and outputs where applicable.

## IAM cleanup

1. If an administrator granted a lab-only permission specifically for this exercise, ask them to remove it or allow its approved temporary session to expire.
2. If a dedicated lab-only role was explicitly created, remove only that role and its attached lab-only policies after confirming it has no other consumers. This is not expected for the default lab path.
3. Do not remove the learner's pre-existing sandbox profile, IAM Identity Center assignment, role, or shared policy. If the grant cannot safely be identified as lab-only, leave it in place and ask the administrator to review it.

## AWS verification

- Confirm there is no lab-specific CloudFormation stack, Lambda function, Comprehend endpoint, Comprehend job, or S3 bucket; none should have been created.
- Confirm the live API call count is within the approved limit. Billing/usage views can lag; check the account's normal billing monitor after its reporting delay.
- Record whether any temporary permission was removed/expired and who verified it. Do not represent delayed billing data as immediate proof of zero cost.

## Completion record

- Local scratch files removed: yes / no / not applicable
- Persistent AWS resources created: expected none; actual:
- Lab-only permission removed or expired: yes / no / not applicable
- Usage/cost reviewed (and date):
- Cleanup verified by:
