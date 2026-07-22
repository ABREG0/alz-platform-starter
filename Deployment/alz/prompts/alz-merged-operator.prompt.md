---
description: "Operate the full Azure Landing Zone generation-to-deployment loop for this repo. Use when: ALZ scaffold, workflow generation, Terraform generation, staging, promotion, preflight, validation, deploy planning, or triage."
name: "ALZ Merged Operator"
argument-hint: "Describe the ALZ task, mode, target environment, and constraints"
agent: "agent"
---

Run an efficient Azure Landing Zone operator workflow for this repo.

Use the slash-command argument as the task request. If the request is ambiguous, ask only the minimum follow-up questions required to proceed safely.

Always ground the work in these repo assets:
- [Platform reproduction guide](../../Deployment/1-file-to-give-to-AI.md)
- [ALZ deploy runbook](../../Deployment/alz/deploy-runbook.md)
- [ALZ artifact pack](../../Deployment/alz/README.md)
- [ALZ generation prompt](../../Deployment/alz/generate-alz-environment.prompt.md)
- [ALZ full multi-region repro prompts](../../terraform/environments/dev/alz-full-multi-region/_docs_scripts/alz-full-multi-region-repro-prompts.md)
- [ALZ full multi-region stack](../../terraform/environments/dev/alz-full-multi-region/main.tf)

Supported modes:
- `generate`: create workflows, scripts, Terraform, and tfvars from a platform spec
- `stage`: write generated outputs under `Deployment/alz/generated`
- `review`: compare staged outputs to existing repo patterns and identify deltas
- `preflight`: verify repo, workflow, OIDC, backend, and subscription prerequisites
- `validate`: run the smallest checks that can falsify the current implementation
- `deploy`: execute the next deployment-safe commands or workflows when prerequisites exist
- `triage`: classify and repair a workflow or Terraform failure

Default recommended profile (use only when user does not provide explicit values and policy allows defaults):
- `caf_alignment_profile`: `enterprise`
- `waf_prioritized_pillars`: `security`, `reliability`, `operational_excellence`, `cost_optimization`, `performance_efficiency`
- `subscription_vending_strategy`: `avm-subscription-module`
- `subscription_lifecycle_owner`: `platform-team`
- `subscription_move_policy`: `create-only`
- `target_management_group_id`: required (do not invent)
- `avm_subscription_module_version`: required (do not invent)
- naming contract values are required (do not invent workload, environment, region, instance, uniqueness, abbreviation, or exception values)

Required operating rules:
- Inputs first. Extract or request the platform spec before generation.
- Always collect and validate CAF/WAF governance inputs before generation or deployment:
	- `caf_alignment_profile`
	- `waf_prioritized_pillars` (all five pillars, unique, ordered)
- Do not invent tenant IDs, subscription IDs, backend names, GitHub environment names, or approval boundaries.
- Always collect and validate subscription vending inputs before generation or deployment:
	- `subscription_vending_strategy` (`none`, `msft-vending-workflows`, `avm-subscription-module`, `hybrid`)
	- `subscription_lifecycle_owner`
	- `subscription_move_policy` (`create-only`, `create-and-move`)
	- `avm_subscription_module_version` when AVM strategy is selected
- Always collect and validate the PLATFORM SPEC resource naming contract before generation or deployment:
	- workload and environment codes
	- three-digit instance number
	- global uniqueness suffix
	- primary and secondary region short codes
	- resource type abbreviations and resource-specific patterns
	- approved naming exceptions
- Generate names from centralized Terraform locals. Do not hardcode or independently concatenate resource names in modules.
- Enforce lowercase, component formats, and Azure resource-specific length and character rules.
- Check Azure name availability for newly generated globally unique names before plan and stop for a new suffix on collision.
- Stop before generation when any naming input or composed name is missing or invalid.
- Stop on a nonconforming supplied name unless its exact name, resource scope, and reason are recorded as an approved exception.
- Never rename existing management group IDs or referenced external resources.
- Prefer existing repo structures, naming, and workflow patterns over generic Azure examples.
- This is a prompt test. Keep every generated or derived artifact under `Deployment/alz` only.
- Do not create, update, or propose active-root promotion outside `Deployment/alz` unless the user explicitly changes that constraint.
- Always include ALZ management group hierarchy deployment in ALZ scope work.
- Always include role assignments at management group scope in ALZ scope work.
- Always include ALZ policy deployment plus AMBA onboarding/configuration in ALZ scope work.
- Never move subscriptions into management groups unless the user explicitly requests subscription moves.
- If `subscription_move_policy` is `create-only`, block any step that attempts to move existing subscriptions.
- After the first substantive edit, run the narrowest available validation before widening scope.
- Continue automatically after generation; do not stop at file creation when validation or execution can proceed safely.
- Before any external execution step, check whether Azure access, GitHub access, secrets, repo variables, and approval paths are available.
- If access is missing, ask the user for it or provide the exact prerequisite action before continuing.

Execution sequence:
1. Determine mode.
2. Identify the owning repo surface.
3. State assumptions and blockers.
4. Validate naming inputs and compute the expected resource names.
5. Make the smallest grounded change or generation step.
6. Run focused validation, including computed-name checks.
7. If generated-artifact validation fails, repair the same slice and rerun; invalid or missing PLATFORM SPEC inputs must be returned to the user rather than repaired or invented.
8. If validation succeeds and prerequisites exist, execute the next safe step automatically.
9. Only then continue to adjacent follow-up work.

Output in this exact order:

## Mode
- State the selected mode and why.

## Inputs
- List the concrete values used.
- List missing required values if any.

## Plan
- Give the shortest safe execution path.

## Changes
- List files or surfaces created, staged, promoted, or updated.

## Validation
- Show the exact checks run or the precise checks still required.

## Risks and Blockers
- List only real blockers, conflicts, or safety constraints.
- Explicitly separate missing access, missing secrets, and missing approvals.

## Next Action
- State the single best next step.

If the request is to generate files, produce complete file content for each required file.
If the request is to deploy, continue automatically until a real external blocker is encountered.
