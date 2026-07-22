# Deploy Runbook

This runbook is the execution path after the ALZ artifacts are generated under `./Deployment/alz/generated`.

For a visual high-level map of what the prompts generate and deploy, open [alz-prompt-deployment-route-explorer.html](./alz-prompt-deployment-route-explorer.html) first.

## 1. Fill the spec

Complete [platform-spec.template.md](./platform-spec.template.md).

Stop if any of these are missing:

- Entra tenant ID
- four ALZ subscription IDs
- backend storage account and resource group
- GitHub runner group and label
- GitHub environment names
- workload and environment naming codes
- three-digit instance number and global uniqueness suffix
- short codes for both regions
- resource type abbreviations and resource-specific patterns
- an explicit naming exception list

## 2. Generate the artifacts

Use [generate-alz-environment.prompt.md](./generate-alz-environment.prompt.md) with the completed spec.

Expected outputs:

- `./Deployment/alz/generated/.github/workflows/*`
- `./Deployment/alz/generated/scripts/*`
- `./Deployment/alz/generated/terraform/*`

## 3. Review against the repo's working ALZ stack

Compare the generated output to these repo anchors:

- [../../terraform/environments/dev/alz-full-multi-region/main.tf](../../terraform/environments/dev/alz-full-multi-region/main.tf)
- [../../terraform/environments/dev/alz-full-multi-region/variables.tf](../../terraform/environments/dev/alz-full-multi-region/variables.tf)
- [../../terraform/environments/dev/alz-full-multi-region/run-platform-alz-prod.ps1](../../terraform/environments/dev/alz-full-multi-region/run-platform-alz-prod.ps1)

Focus on:

- `subscription_ids` alignment
- backend authentication model
- Terraform and provider versions
- environment naming for plan/apply workflows
- centralized naming locals, compliant computed names, and exact approved exceptions

## 4. Validate locally before promotion

From repo root, validate the existing ALZ root:

```powershell
terraform -chdir=terraform/environments/dev/alz-full-multi-region fmt -recursive
terraform -chdir=terraform/environments/dev/alz-full-multi-region init -input=false -reconfigure
terraform -chdir=terraform/environments/dev/alz-full-multi-region validate
```

Before planning, confirm generated names are lowercase and contract-compliant; Storage, Key Vault, and ACR must also pass their tighter length, character, and global availability checks, or use a new suffix.

If backend access is already configured, use the local wrapper for a safer environment-specific plan:

```powershell
./terraform/environments/dev/alz-full-multi-region/run-platform-alz-prod.ps1 -PlanOnly
```

## 5. Keep everything staged for prompt testing

For this test, do not move or adapt generated files into active repo locations.

Keep everything under `Deployment/alz/generated`, including:

- `.github/workflows`
- `scripts`
- `terraform`
- `optfiles`

## 6. Azure and GitHub prerequisites

Before any real deployment:

- ensure the service principal has federated credentials for the target environments
- ensure the required GitHub Environments exist
- ensure backend storage resources exist or allow the bootstrap workflow to create them
- ensure the operator or workflow identity has required RBAC on target subscriptions and backend storage

## 7. Deployment reality check

This folder prepares and stages the ALZ deployment artifacts.

It does not execute a live Azure deployment by itself because that requires:

- filled environment values
- Azure authentication context
- an explicit decision to move beyond the prompt-test staging area
