You are onboarding a new application team into this ALZ platform.

## Inputs (required)

- `team_name`
- `application_name`
- `application_abbreviation`
- `target_environment` (`dev` | `prod`)
- `app_subscription_id`
- `connectivity_subscription_id`
- `identity_subscription_id`
- `security_management_subscription_id`
- `requested_regions`
- `requested_cidrs` (vnet + subnets)
- `required_endpoints` (web app, storage, key vault)
- `go_live_date`
- `caf_alignment_profile` (`foundational` | `enterprise` | `regulated`)
- `waf_prioritized_pillars` (ordered list from `reliability`, `security`, `cost_optimization`, `operational_excellence`, `performance_efficiency`)
- `subscription_vending_strategy` (`none` | `msft-vending-workflows` | `avm-subscription-module` | `hybrid`)
- `subscription_lifecycle_owner` (`platform-team` | `app-team` | `central-it`)
- `subscription_move_policy` (`create-only` | `create-and-move`)
- `avm_subscription_module_version` (required when strategy is `avm-subscription-module` or `hybrid`)
- `target_management_group_id`

## Onboarding Objectives

1. Confirm technical and governance readiness.
2. Identify blockers before deployment execution.
3. Produce an approved Terraform input pack.
4. Approve CAF/WAF posture and subscription vending path before provisioning.

## Default Recommended Profile
Use this profile only when the user does not provide explicit values and policy allows defaults:
- `caf_alignment_profile`: `enterprise`
- `waf_prioritized_pillars`: `security`, `reliability`, `operational_excellence`, `cost_optimization`, `performance_efficiency`
- `subscription_vending_strategy`: `avm-subscription-module`
- `subscription_lifecycle_owner`: `platform-team`
- `subscription_move_policy`: `create-only`
- `target_management_group_id`: required (do not invent)
- `avm_subscription_module_version`: required (do not invent)

## Mandatory Readiness Checks

1. CIDRs are non-overlapping with hub ranges and reserved ranges.
2. Naming follows `application_abbreviation` convention.
3. Role assignments are complete for:
   - app deployment scope
   - both sides of cross-subscription peering
   - private DNS zone link/record operations
   - Terraform backend state read/write/lock
4. Central private DNS zones exist and are accessible:
   - `privatelink.azurewebsites.net`
   - `privatelink.blob.core.windows.net`
   - `privatelink.vaultcore.azure.net`
5. Backend state key is unique and environment-aligned.
6. CAF profile and WAF pillar priority are explicitly approved and recorded.
7. Subscription vending decision is explicit and executable:
   - strategy selected (`none`, `msft-vending-workflows`, `avm-subscription-module`, or `hybrid`)
   - lifecycle owner selected
   - move policy selected (`create-only` or `create-and-move`)
   - AVM module version present when AVM strategy is selected
   - target management group is specified

## Stop Conditions

- Missing subscription IDs, CIDRs, or required role scopes.
- CIDR overlap detected.
- Central private DNS dependency missing.
- Missing CAF/WAF approvals.
- Subscription vending inputs are incomplete or conflict with move policy.

## Output Contract (exact order)

1. `READINESS_STATUS` (`READY` | `BLOCKED`)
2. `BLOCKERS_CHECKLIST`
3. `APPROVED_TFVARS_INPUT_PACK`
4. `REQUIRED_ROLE_ASSIGNMENTS`
5. `NEXT_5_EXECUTION_STEPS`