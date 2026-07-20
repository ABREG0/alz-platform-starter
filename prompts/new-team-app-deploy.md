You are executing a new team app landing-zone deployment with Terraform in this repository.

## Inputs (required)

- `target_environment` (`dev` | `prod`)
- `app_tfvars_path`
- `app_backend_hcl_path`
- `legacy_state_key`
- `run_apply` (`true` | `false`)
- `run_destroy_after_validation` (`true` | `false`)

## Deployment Scope

- app VNet + workload subnets
- Linux Web App + App Service plan
- Storage Account
- Key Vault
- Private Endpoints (web app, blob, key vault)
- Central private DNS integration
- Linux and Windows VM
- Bidirectional peering to legacy hubs

## Execution Sequence

1. Validate required inputs and reject placeholders.
2. Run `terraform fmt -recursive`.
3. Run `terraform init` with provided backend config.
4. Run `terraform validate`.
5. Run `terraform plan -out=tfplan`.
6. If `run_apply=true`, run `terraform apply tfplan`.
7. Validate expected outputs (vnet, private endpoints, vm ids, legacy hub ids).
8. Execute smoke checks for peering and DNS resolution.
9. If `run_destroy_after_validation=true`, run `terraform destroy -auto-approve`.

## Gates (stop immediately)

- Validation or plan failure.
- Missing central DNS zones.
- Missing peering permissions on either side.
- Apply requested but plan contains unapproved destructive changes.

## Output Contract (exact order)

1. `FILES_USED`
2. `COMMANDS_EXECUTED`
3. `STEP_RESULTS` (`PASS` | `FAIL` per step)
4. `BLOCKING_ISSUES`
5. `REMEDIATION_ACTIONS`