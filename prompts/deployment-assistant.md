You are implementing Terraform changes in this repository.

## Inputs (required)

- `change_scope`: exact behavior to add/change/remove
- `deployment_type`: `connectivity` | `vwan-connectivity` | `selfhosted-aks` | `vnet-integrated-runners` | `both`
- `environments`: `dev` | `prod` | `both` | `n/a` (for runner roots)
- `target_roots`: explicit list of root folders to touch
- `apply_allowed`: `true` | `false`
- `parity_required`: `true` | `false`
- `docs_update_required`: `true` | `false`

## Non-Negotiable Constraints

- Keep OIDC/AzureAD backend auth pattern.
- Treat `terraform.tfvars` and `backend.hcl` as active files.
- Keep changes minimal, reversible, and scoped to `change_scope`.
- Do not create dev/prod drift unless requested.
- Do not run apply unless `apply_allowed=true`.
- Region short-code naming must be preserved (`eastus` -> `eus`, `eastus2` -> `eus2`, `centralus` -> `cus`).
- DNS placement model must be preserved:
   - full centrally managed DNS set in primary DNS RG
   - region-filtered DNS set in secondary DNS RG
- Validation of DNS existence/link health must use Azure resource queries, not only plan/state output.

## Execution Order

1. **Scope confirmation**
   - List target roots and expected changed files before editing.
2. **Implementation**
   - Update variables and locals before module wiring/resources.
   - Preserve naming and tagging conventions already in repo.
3. **Static quality**
   - Run `terraform fmt -recursive` in each target root.
   - Run `terraform init -backend=false` in each target root.
   - Run `terraform validate` in each target root.
4. **Plan/apply**
   - Run `terraform plan -out=tfplan` in each target root.
   - If `apply_allowed=true`, run `terraform apply tfplan`.
5. **Operational checks** (for env roots)
   - `pwsh -File ./scripts/smoke-check.ps1 -EnvironmentPath <path>`
   - `pwsh -File ./scripts/validate-deployment.ps1 -EnvironmentPath <path>`
   - Azure query validation:
     - `az network private-dns zone list -g <primary-dns-rg>`
     - `az network private-dns zone list -g <secondary-dns-rg>`
     - verify `numberOfVirtualNetworkLinks > 0` for all expected zones
6. **Docs/prompts sync**
   - If behavior or required inputs changed, update related docs/prompts in same change set.

## Gates (stop immediately)

- Missing required input values.
- `terraform validate` fails in any target root.
- `terraform plan` errors in any target root.
- Unexpected destructive changes appear in plan (unless explicitly approved).
- Drift detected in untouched resources without user approval.
- Any DNS zone expected by config is missing in Azure after apply.
- Any expected DNS zone has zero VNet links in Azure after apply.

## Required Output Contract

Return only these sections, in this order:

1. `FILES_CHANGED`
2. `COMMANDS_EXECUTED`
3. `VALIDATION_SUMMARY`
4. `PLAN_SUMMARY`
5. `RISKS_AND_FOLLOW_UP`

## DNS Validation Snippet (Azure-first)

Use this PowerShell snippet to detect no-link zones:

`$zones = az network private-dns zone list -g <dns-rg> -o json | ConvertFrom-Json`

`$zones | Where-Object { $_.numberOfVirtualNetworkLinks -eq 0 } | Select-Object name, numberOfVirtualNetworkLinks`
