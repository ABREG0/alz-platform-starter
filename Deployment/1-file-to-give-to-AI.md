# Prompts to Give an AI — Reproduce This Platform in a New Environment

This guide answers two questions:

1. **Can an AI regenerate and execute the platform workflow from prompts alone?**
   Yes, as long as the environment-specific decisions and required access are provided. You supply the values once. The AI should generate the code, stage the artifacts, validate them, and execute the next safe workflow or command automatically.

2. **Do I need to copy files to another environment to reproduce this?**
   No. You give the AI a short spec block (a single text section with your decisions), and it generates workflows, Terraform scaffold, modules, bootstrap script, `directory-config.json`, tfvars structure, and then proceeds through validation and execution automatically when access and approvals allow.

---

## How it works

```text
You provide  →  a spec block with your decisions (names, IDs, roles, regions)
AI generates →  all workflow files, Terraform root, modules, bootstrap script, directory-config.json
AI checks    →  Azure access, GitHub access, secrets, approvals, and required execution context
AI validates →  generated files, Terraform shape, workflow shape, and prerequisite alignment
AI executes  →  the next safe command or workflow automatically, stopping only for missing access, approvals, or external blockers
```

The only things you must decide in advance are listed in the spec template below. Everything else should be handled by the AI unless an external permission, approval, missing secret, or missing access context blocks execution. When access is missing, the AI should explicitly ask for it or tell the user exactly what must be provided.

---

## Step 1 — Fill in your spec (the only thing you write)

Copy this block, fill in your values, and paste it into the AI prompt in Step 2.

```text
PLATFORM SPEC
=============

## Identity
Entra tenant ID: 72f988bf-86f1-41af-91ab-2d7cd011db47
Service principal display name:  alz-ai
Service principal client ID:  c484d43d-b449-4cee-aae3-077a3a4246b5

## Subscriptions
Management subscription ID: ee5d2fa3-deed-4b33-bc7c-b5fbe997bc65
Connectivity subscription ID: 764741ea-c5b3-4b04-af32-62001793d88e
Identity subscription ID: 6284f04c-ec26-45e3-a7a6-24c2ef4722e4
Security subscription ID: c5c1228d-b650-4f0a-97ea-1f8cfdc417c5
Spoke subscription IDs: c5c1228d-b650-4f0a-97ea-1f8cfdc417c5, 6284f04c-ec26-45e3-a7a6-24c2ef4722e4

## State backend
Storage account name: stb7q2znqmpox3hmix
Resource group name: rg-tfstate-shared
subscription ID for state storage: c5c1228d-b650-4f0a-97ea-1f8cfdc417c5
Region: eastus2
Blob container for state:           tfstate
Blob container for plans:           tfplan

## GitHub
Organization / Enterprise name:     ABREG0
Repo name(s): alz1 , alz2   # one entry per IaC repo
Runner group name: my-org-runners-prd
Runner label: aks-arc-runners-prd
GitHub Environment for plan:        prd
GitHub Environment for apply:       prd-Apply
Apply approver team:  my-org-devlead-team

## Terraform
Terraform version:                  ~> 1.12
AzureRM provider version:           ~> 4.0

## Regions
Primary region: centralus
Secondary region: southcentralus

## RBAC roles to assign on each spoke subscription
- Contributor
- Storage Blob Data Contributor

## RBAC roles to assign on state storage resource group (bootstrap only)
- Storage Blob Data Owner
- Storage Account Contributor
```

---

## Step 2 — Prompt to generate, validate, and execute everything

Paste your filled-in spec from Step 1, then add this prompt:

```text
Using the PLATFORM SPEC above as the only source of values, generate the following files
for a new Azure Landing Zone IaC platform using Terraform and GitHub Actions.
Do not invent values — use only what is in the spec.
After generation, validate the files and proceed automatically to the next safe execution step.
Stop only when a required approval, missing secret, missing external permission, or unavailable tool blocks further progress.

Generate:

1. `.github/workflows/Bootstrap.yaml`
   - workflow_dispatch trigger, no inputs
   - OIDC login using Azure/login@v2 with vars.AZURE_CLIENT_ID / AZURE_TENANT_ID
   - Creates the state storage resource group and storage account (RA-GRS, TLS 1.2, no shared keys, no public blob)
   - Creates blob containers: tfstate and tfplan
   - Assigns Storage Blob Data Owner and Storage Account Contributor to the service principal on the storage RG
   - Runs on the runner group and label from the spec
   - Uses GitHub Environment: prd

2. `.github/workflows/pr-validate.yml`
   - Triggers on pull_request to main
   - Detects which directories changed by reading terraform/directory-config.json
   - Runs terraform fmt, validate, plan in a matrix per changed directory
   - Injects backend config from GitHub Actions vars (no values hardcoded)
   - Posts plan output as a PR comment
   - Runs on the runner group and label from the spec
   - Uses GitHub Environment: prd

3. `.github/workflows/terraform-deploy.yml`
   - workflow_dispatch with inputs: release_tag (required), working_directory (optional override)
   - Enforces main branch
   - Detects directories to deploy from diff between release_tag and previous tag, reading directory-config.json
   - Splits into plan job (environment: prd) and apply job (environment: prd-Apply, needs manual approval)
   - Runs on the runner group and label from the spec
   - Uploads plan as artifact; apply downloads and executes it
   - Injects backend config from vars

4. `.github/workflows/release.yml`
   - Triggers on push to main
   - Auto-increments a semver tag (v1.0.0 → v1.0.1) and creates a GitHub release

5. `scripts/setup-federated-creds-and-rbac.ps1`
   - PowerShell script, params: ServicePrincipalClientId, Repository, EnvironmentNames, SubscriptionIds, RoleNames
   - Creates OIDC federated credentials on the Entra app for each environment and for pull_request
   - Assigns the roles from the spec to the SP on each subscription
   - Uses az CLI; no hardcoded values — all driven by parameters

6. `terraform/directory-config.json`
   - JSON with: terraform_version, directories object (empty for now — user populates entries)
   - Include one example entry showing the schema: path, state_key, tfvar_args

7. `terraform/spoke-unified/prod/terraform.tf`
   - required_version and azurerm provider version from spec
   - Empty azurerm backend block

8. `terraform/spoke-unified/prod/providers.tf`
   - azurerm provider with features {} and subscription_id from var.default_subscription_id

9. `terraform/spoke-unified/prod/variables.tf`
   - variable default_subscription_id
   - variable spoke_components: map of objects supporting network, nsg, and route-table component types
     with: enabled, component_type, subscription_id, location, resource_group_name, tags,
     virtual_network_name, virtual_network_address_space, dns_servers, subnets map,
     nsg_name, nsg_names map, security_rules list, nsg_subnet_associations list,
     route_table_name, bgp_route_propagation_enabled, routes list, route_table_subnet_associations list

10. `terraform/spoke-unified/prod/main.tf`
    - locals to filter spoke_components by component_type and enabled flag
    - module "spoke_network" for_each over network components
    - module "spoke_nsg" for_each over nsg components
    - module "spoke_route_table" for_each over route-table components

11. `terraform/spoke-unified/prod/modules/spoke-network/main.tf` and `variables.tf`
    - azurerm_virtual_network resource
    - azurerm_subnet resource for_each over var.subnets

12. `terraform/spoke-unified/prod/modules/spoke-nsg/main.tf` and `variables.tf`
    - supports nsg_names map (one NSG per subnet key) or falls back to single nsg_name
    - azurerm_network_security_group for_each
    - azurerm_network_security_rule for_each over security_rules cross-joined with NSGs
    - data azurerm_subnet lookup and azurerm_subnet_network_security_group_association

13. `terraform/spoke-unified/prod/modules/spoke-route-table/main.tf` and `variables.tf`
    - azurerm_route_table
    - azurerm_route for_each over var.routes
    - data azurerm_subnet lookup and azurerm_subnet_route_table_association

14. `terraform/spoke-unified/prod/tfvars/prod.tfvars`
    - One example enabled network component and one disabled placeholder showing the full schema

For each file, output the complete file content. Do not abbreviate or use placeholders
like "# ... rest of file". Every file must be complete and immediately usable.

After generating the files, the AI must also:

1. Validate the generated Terraform and workflow structure.
2. Check whether Azure access, GitHub access, required secrets, and approval paths are available.
3. If access is missing, ask the user for the missing access or state exactly what must be configured.
4. Create or stage the files in the target folder.
5. Run the smallest safe validation commands immediately.
6. If GitHub or Azure execution is available, proceed with:
   - OIDC and backend prerequisite checks
   - federated credential and RBAC setup
   - bootstrap workflow or equivalent backend creation step
   - Terraform plan or deployment workflow dispatch
7. Report only the blockers that require human action.
```

---

## Step 3 — Automatic execution rule

After the AI generates the files, it should continue automatically through validation and execution.

The AI should perform these steps itself whenever tooling and access are available:

| Action | Expected executor |
| --- | --- |
| Create or verify GitHub Environments `prd` and `prd-Apply` | AI, if GitHub access is available |
| Verify or report required reviewer on `prd-Apply` | AI |
| Verify or report repo vars: `AZURE_CLIENT_ID`, `AZURE_TENANT_ID`, `AZURE_SUBSCRIPTION_ID` | AI |
| Run `setup-federated-creds-and-rbac.ps1` | AI |
| Run `Bootstrap.yaml` workflow or equivalent backend creation path | AI |
| Run Terraform validation, plan, and next deployment-safe action | AI |

Before any execution step, the AI should check and report:

- whether Azure authentication is available
- whether GitHub workflow or repo access is available
- whether required repo variables or secrets are already configured
- whether any approval gate requires human action

If one of those is missing, the AI should prompt the user for the missing access or provide the exact configuration action needed before continuing.

The AI should stop only for real external blockers such as:

- missing Azure authentication
- missing GitHub permissions
- missing secrets or repo variables
- required human approval gates
- unavailable tools or service-side failures
