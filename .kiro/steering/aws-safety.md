# AWS safety steering

- Default to `ap-southeast-1` when the required service/features are available; verify availability rather than assume it.
- Prefer short-lived, low-cost sandbox resources and disclose likely cost drivers before provisioning.
- Never deploy, create, modify, or delete billable resources without explicit learner approval.
- Use a named sandbox identity and standard credential providers. Never put credentials in source, specs, logs, or committed environment files.
- Prefer least privilege, encryption, and observable failure handling in all lab requirements.
- Include cleanup steps and post-cleanup verification for every lab with provisioned resources.
- For AWS claims likely to change, consult official documentation or the configured AWS MCP and cite the source in the relevant artifact.
