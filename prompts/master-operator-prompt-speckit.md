# SPEC-IT: ALZ Terraform + Workflow Master Operator

## Objective
Execute one deterministic infrastructure operation (`workflow-deploy`, `local-deploy`, `validate`, or `troubleshoot`) for ALZ Terraform roots with Azure-first verification.

## Continuous Optimization Mandate
- Always optimize the prompt and spec artifacts when new execution evidence is available.
- After each run, capture and fold back:
  - newly observed failure signatures,
  - safer command ordering,
  - stricter input validation,
  - improved output contract clarity.
- Do not leave optimization as optional follow-up; treat it as part of completion.

## Required Inputs
- `mode`: workflow-deploy | local-deploy | validate | troubleshoot
- `environment`: dev | prod
- `deployment_type`: connectivity | vwan-connectivity | selfhosted-aks | vnet-integrated-runners | app-landing-zone
- `target_root`: explicit path
- `apply_allowed`: true | false
- `parity_required`: true | false
- `caf_alignment_profile`: foundational | enterprise | regulated
- `waf_prioritized_pillars`: ordered list from reliability, security, cost_optimization, operational_excellence, performance_efficiency
- `subscription_vending_strategy`: none | msft-vending-workflows | avm-subscription-module | hybrid
- `subscription_lifecycle_owner`: platform-team | app-team | central-it
- `subscription_move_policy`: create-only | create-and-move
- `avm_subscription_module_version`: required when `subscription_vending_strategy` is `avm-subscription-module` or `hybrid`
- `change_scope`: required for deploy
- `error_snippet`: required for troubleshoot
- `workflow_name`: required for `workflow-deploy` (example: `advanced-workflows-alz-full-test.yml`)
- `workflow_action`: apply | destroy | both (required for `workflow-deploy`)
- `destroy_approved`: yes | no (required when `workflow_action` includes `destroy`; must be `yes` to execute destroy)
- `optfile_path`: required for `workflow-deploy`

## Non-Functional Requirements
- Preserve OIDC + Azure AD backend auth.
- Preserve dev/prod parity unless explicitly overridden.
- Preserve workflow reusable chain compatibility:
  - `pipeline-apply.yml` / `pipeline-destroy.yml`
  - `read-file.yml`, `terraform-fmt-check.yml`, `terraform-validate.yml`, `terraform-plan.yml`, `terraform-apply.yml`, `terraform-destroy.yml`
- Preserve naming and placement conventions:
  - short codes: eastus=eus, eastus2=eus2, centralus=cus
  - DNS model: full central set in primary RG, regional set in secondary RG
- Preserve governance intent encoded in inputs:
  - CAF profile selection must be explicit for each run.
  - WAF pillar priority order must be explicit for each run.
  - Subscription vending decisions must be explicit before deploy activity.
- Use Azure resource queries as source of truth for DNS validation.

## Default Recommended Profile
Use this profile only when the user does not provide explicit values and policy allows defaults:
- `caf_alignment_profile`: enterprise
- `waf_prioritized_pillars`: security, reliability, operational_excellence, cost_optimization, performance_efficiency
- `subscription_vending_strategy`: avm-subscription-module
- `subscription_lifecycle_owner`: platform-team
- `subscription_move_policy`: create-only
- `target_management_group_id`: required (do not invent)
- `avm_subscription_module_version`: required (do not invent)

## Functional Specification

### 1) Workflow Deploy Flow (Primary)
- Dispatch workflow with explicit inputs: `action`, `destroy_approved`, `optfile_path`, `environment_name`, `azure_client_id`, `azure_tenant_id`, `ref`.
- Before dispatch, validate governance/vending inputs:
  - `caf_alignment_profile` is declared.
  - `waf_prioritized_pillars` includes all five pillars with no duplicates.
  - `subscription_vending_strategy`, `subscription_lifecycle_owner`, and `subscription_move_policy` are declared.
  - If AVM subscription vending is selected, `avm_subscription_module_version` is present.
- Verify stages pass in sequence:
  1. `read_optfile`
  2. `fmt`
  3. `validate`
  4. `plan`
  5. `apply` (if requested)
  6. `destroy` (if requested)
- If workflow fails, classify by OIDC subject mismatch, backend auth mode, missing backend resources, missing secrets, or terraform errors.
- Include vending failure classification when relevant:
  - missing/invalid AVM subscription module version
  - missing role assignments for vending identity
  - management group move denied by policy (`create-only`)
- Include backend network model checks (`publicNetworkAccess`, private endpoint path) when backend auth appears correct but state access still fails.

### 2) Local Deploy Flow (Fallback)
- Apply minimal changes scoped to `change_scope`.
- Run: fmt, init (backend=false), validate, plan.
- Apply only if `apply_allowed=true`.
- For connectivity, verify DNS via Azure queries:
  - enumerate zones in primary/secondary DNS RG
  - verify expected counts
  - verify no zones with `numberOfVirtualNetworkLinks = 0`

### 3) Validate Flow
- Run root validate.
- Run repository validation scripts where applicable.
- Run Azure DNS verification queries and record evidence.

### 4) Troubleshoot Flow
- Derive primary root cause from `error_snippet`.
- Evaluate likely sources in priority order:
  1. OIDC/workflow environment subject mismatch (`<env>`, `<env>-Apply`, `<env>-Destroy`)
  2. Backend config or storage RBAC
  3. Backend network reachability (`publicNetworkAccess=Disabled` without private path)
  4. Subscription placement mismatch
  5. DNS zone/link drift in Azure
- Produce minimal remediation and verification sequence.
- Include vending-specific remediation when `subscription_vending_strategy` is not `none`.

Recommended backend remediation command (preview first):
- `./scripts/grant-backend-blob-rbac.ps1 -SubscriptionId <subId> -ResourceGroupName <rg> -StorageAccountName <storage-account> -ContainerName tfstate`

## Acceptance Criteria
- No missing required inputs.
- Successful `terraform validate` for target root.
- If `workflow-deploy`, target workflow run reaches requested end state.
- Destroy operations execute only when `destroy_approved=yes` is explicitly provided.
- If local deploy, plan/apply succeed for approved scope.
- Azure verification confirms:
  - expected DNS zones are present
  - no DNS zone has zero VNet links
- Output uses required contract order and contains actionable commands.
- Prompt/spec optimization deltas are recorded when new behavior is discovered.
- CAF/WAF/vending inputs are present and validated before deploy execution.

## Output Contract
1. MODE
2. FILES_CHANGED
3. COMMANDS_EXECUTED
4. VALIDATION_AND_AZURE_RESULTS
5. RISKS_AND_NEXT_ACTIONS
