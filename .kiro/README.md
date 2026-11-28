# Kiro-style workflow in OpenCode

This repository keeps Kiro's useful workflow concepts in portable Markdown and OpenCode-native configuration:

- **Steering:** `.kiro/steering/` contains durable project, learner, and AWS guardrails. `opencode.json` loads these files as ambient instructions.
- **Specs:** `.kiro/specs/<change>/` contains requirements, design, and tasks. Start one with `/specify <change>`; review it with `/spec-review <folder>`.
- **Agents:** `.opencode/agents/` defines the default AWS lab coach, spec planner, and read-only spec reviewer.
- **Skills:** `.opencode/skills/` holds reusable workflow instructions that load only when relevant.
- **Commands:** `.opencode/commands/` exposes `/lab`, `/assess`, `/specify`, and `/spec-review`.
- **Tasks:** OpenCode's experimental todo tool is enabled to track multi-step work during a session.

## Starting a change

1. For learner lab work, run `/lab`; the coach will resume the active curriculum step and keep implementation learner-owned.
2. For a project/tooling feature spanning multiple files, run `/specify <plain-language outcome>`.
3. Review the generated package with `/spec-review <folder-name>` and resolve blocking findings.
4. Ask explicitly for implementation when ready. The agent should follow the project's learner-ownership and AWS provisioning gates.
5. Verify each task against observable evidence, update `tasks.md`, and capture outcomes in the appropriate project artifact.

## Templates

Copy the three files in `.kiro/specs/_template/` into a new spec folder. Keep change-specific decisions in that spec; keep recurring rules in `.kiro/steering/`.

## MCP and skills

The repository inherits the user's global OpenCode integrations rather than duplicating credentials or machine-specific commands. At setup time, the global MCP inventory was connected: `aws-mcp`, `codegraph`, `ollaya`, `openscreen`, and `iwork`. Check current status with `opencode mcp list`.

The global AWS and developer skill catalogs are also discovered by OpenCode. This repo adds only `spec-driven-development`; do not copy global skills into the project. AWS MCP access is for documentation and read-only inspection by default. Creating, changing, or deleting billable AWS resources still requires explicit learner approval.
