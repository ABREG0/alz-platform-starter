You are performing post-deployment validation and operational handoff for a new team app deployment.

## Inputs (required)

- `environment`
- `app_resource_group_name`
- `app_vnet_name`
- `linux_vm_name`
- `windows_vm_name`
- `web_app_name`
- `storage_account_name`
- `key_vault_name`
- `evidence_output_path`
- `caf_alignment_profile` (`foundational` | `enterprise` | `regulated`)
- `waf_prioritized_pillars` (ordered list from `reliability`, `security`, `cost_optimization`, `operational_excellence`, `performance_efficiency`)
- `subscription_vending_strategy` (`none` | `msft-vending-workflows` | `avm-subscription-module` | `hybrid`)
- `subscription_lifecycle_owner` (`platform-team` | `app-team` | `central-it`)
- `subscription_move_policy` (`create-only` | `create-and-move`)
- `avm_subscription_module_version` (required when strategy is `avm-subscription-module` or `hybrid`)
- `target_management_group_id` (required when strategy is not `none`)

## Validation Checklist

1. **Infrastructure drift**
   - `terraform plan -detailed-exitcode` returns `0`.
2. **Network and peering**
   - App VNet is peered both directions with required legacy hubs.
3. **Private connectivity**
   - Private endpoints exist for web app, blob, and key vault.
   - Required DNS zone links exist and are healthy.
4. **Compute and platform**
   - Linux VM and Windows VM exist and are running.
   - Web app, Storage Account, and Key Vault have public access disabled.
5. **Security and governance**
   - Mandatory tags exist on all scoped resources.
   - Required role assignments are documented and validated.
6. **Governance decision matrix (CAF/WAF/vending)**
   - Evaluate and report pass/fail for each item:
     - `caf_alignment_profile` declared and approved.
     - `waf_prioritized_pillars` includes all five pillars exactly once.
     - `subscription_vending_strategy` declared and executable.
     - `subscription_lifecycle_owner` declared.
     - `subscription_move_policy` declared and honored by executed actions.
     - `avm_subscription_module_version` present when strategy is AVM or hybrid.
     - `target_management_group_id` present when strategy is not `none`.

## Evidence Rules

- Every check must include command/query + result.
- Any failure must include impact + owner + remediation ETA.
- Save/export evidence references to `evidence_output_path`.

## Output Contract (exact order)

1. `VALIDATION_SUMMARY` (`PASS` | `FAIL`)
2. `EVIDENCE_TABLE` (check | command | result)
3. `GOVERNANCE_DECISION_MATRIX` (item | expected | actual | pass_fail)
4. `DRIFT_AND_SECURITY_FINDINGS`
5. `HANDOFF_NOTES`
6. `DAY2_RECOMMENDATIONS` (max 5)