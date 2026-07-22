# Generate Azure Landing Zone Environment

Use [platform-spec.template.md](./platform-spec.template.md) as the source of truth.

Paste the filled `PLATFORM SPEC` first, then run the prompt below exactly.

```text
Using the PLATFORM SPEC above as the only source of values, generate the following files
for a new Azure Landing Zone environment using Terraform and GitHub Actions.
Do not invent values. If a required value is missing, stop and list the missing fields.

Mandatory ALZ scope in generated output:
- include management group hierarchy deployment artifacts
- include role assignments at management group scope
- include ALZ policy deployment and AMBA deployment artifacts
- do not move subscriptions into the management group hierarchy unless explicitly requested

Mandatory naming behavior:
- treat the `Resource naming` section of the PLATFORM SPEC as the naming contract
- use its declared component order, resource type abbreviations, and resource-specific patterns for every newly generated Azure resource
- validate all naming inputs and composed names before generating any file; if any are missing or invalid, stop and list them
- stop on a nonconforming supplied name unless its exact value, resource scope, and reason appear in `Approved naming exceptions`
- treat management group IDs and referenced existing resource names as immutable; never silently rename them
- derive names once in Terraform locals and reuse those locals instead of rebuilding or hardcoding names in resources and modules

This is a prompt test. Write every generated file under `./Deployment/alz/generated` and preserve this structure exactly.
Do not create, update, or suggest promotion into `.github`, `scripts`, `terraform`, `optfiles`, or any other root outside `Deployment/alz`.
After generation, continue automatically with validation and the next safe execution step inside `Deployment/alz` whenever the environment allows it.
Before any execution step, check whether Azure access, GitHub access, required secrets, and approval paths are available. If not, ask the user for the missing access or describe the exact configuration needed.

Generate:

1. `./Deployment/alz/generated/.github/workflows/Bootstrap.yaml`
   - workflow_dispatch trigger, no inputs
   - OIDC login using azure/login@v3.0.0 with vars.AZURE_CLIENT_ID / AZURE_TENANT_ID
   - creates the state storage resource group and storage account
   - uses the backend names from the spec exactly; validates each against the naming contract or an exact approved exception before creation
   - checks storage account name availability when the account does not already exist and stops with a request for a new suffix on collision
   - creates blob containers: tfstate and tfplan
   - assigns Storage Blob Data Owner and Storage Account Contributor to the service principal on the storage resource group
   - runs on the runner group and label from the spec
   - uses the GitHub Environment for plan from the spec

2. `./Deployment/alz/generated/.github/workflows/pr-validate.yml`
   - triggers on pull_request to main
   - detects which directories changed by reading `terraform/directory-config.json`
   - runs terraform fmt, validate, and plan in a matrix per changed directory
   - injects backend config from GitHub Actions vars with no hardcoded secrets
   - posts the plan output as a PR comment
   - runs on the runner group and label from the spec

3. `./Deployment/alz/generated/.github/workflows/terraform-deploy.yml`
   - workflow_dispatch with `release_tag` and optional `working_directory`
   - enforces main branch
   - detects directories to deploy from diff between release tags using `terraform/directory-config.json`
   - splits into plan and apply jobs
   - uses plan/apply environments from the spec
   - uploads plan as artifact and apply consumes that artifact
   - runs preflight Azure availability checks for newly generated Storage, Key Vault, and ACR names before plan

4. `./Deployment/alz/generated/.github/workflows/release.yml`
   - triggers on push to main
   - increments a semver tag and creates a GitHub release

5. `./Deployment/alz/generated/scripts/setup-federated-creds-and-rbac.ps1`
   - parameters: `ServicePrincipalClientId`, `Repository`, `EnvironmentNames`, `SubscriptionIds`, `RoleNames`
   - creates OIDC federated credentials for each environment and for pull requests
   - assigns the roles from the spec to the service principal on each subscription
   - uses az CLI only

6. `./Deployment/alz/generated/terraform/directory-config.json`
   - includes `terraform_version`
   - includes a `directories` object with one example entry showing `path`, `state_key`, and `tfvar_args`

7. `./Deployment/alz/generated/terraform/spoke-unified/prod/terraform.tf`
   - required_version from the spec
   - azurerm provider version from the spec
   - empty azurerm backend block

8. `./Deployment/alz/generated/terraform/spoke-unified/prod/providers.tf`
   - azurerm provider with `features {}`
   - `subscription_id` sourced from `var.default_subscription_id`

9. `./Deployment/alz/generated/terraform/spoke-unified/prod/variables.tf`
   - `default_subscription_id`
   - typed naming inputs for workload, environment, instance, uniqueness suffix, region short codes, and resource type abbreviations
   - validation blocks for naming component formats and Terraform check/precondition blocks for final composed names
   - `spoke_components` schema supporting `network`, `nsg`, and `route-table` through semantic component keys, not generated-resource name strings

10. `./Deployment/alz/generated/terraform/spoke-unified/prod/main.tf`
    - centralized naming locals used by every generated resource and module
    - filters enabled spoke components by component type
    - calls modules for network, NSG, and route table components

11. `./Deployment/alz/generated/terraform/spoke-unified/prod/modules/spoke-network/main.tf`
12. `./Deployment/alz/generated/terraform/spoke-unified/prod/modules/spoke-network/variables.tf`
13. `./Deployment/alz/generated/terraform/spoke-unified/prod/modules/spoke-nsg/main.tf`
14. `./Deployment/alz/generated/terraform/spoke-unified/prod/modules/spoke-nsg/variables.tf`
15. `./Deployment/alz/generated/terraform/spoke-unified/prod/modules/spoke-route-table/main.tf`
16. `./Deployment/alz/generated/terraform/spoke-unified/prod/modules/spoke-route-table/variables.tf`
17. `./Deployment/alz/generated/terraform/spoke-unified/prod/tfvars/prod.tfvars`
    - includes all naming contract values from the PLATFORM SPEC
18. `./Deployment/alz/generated/terraform/environments/prod/alz-full-multi-region/main.tf`
   - includes ALZ management group hierarchy deployment
   - includes ALZ policy assignment deployment and AMBA onboarding
   - keeps subscription associations as references only unless explicit move request is provided
19. `./Deployment/alz/generated/terraform/environments/prod/alz-full-multi-region/variables.tf`
   - includes management group hierarchy inputs
   - includes policy/AMBA feature toggles and configuration inputs
20. `./Deployment/alz/generated/scripts/setup-mg-scope-rbac.ps1`
   - parameters for management group IDs and principal/object IDs
   - assigns requested roles at management group scope via az CLI

For each file:
- output the complete file content
- do not abbreviate or omit sections
- prefer repo conventions from `terraform/environments/dev/alz-full-multi-region`
- keep authentication aligned with OIDC and Azure AD auth
- do not emit ad hoc resource-name literals when a name can be derived from the naming contract

After generating the files, output these sections in order:
1. GENERATED_FILES
2. ASSUMPTIONS
3. AUTOMATIC_ACTIONS
4. VALIDATION_COMMANDS
5. STAGING_NOTES

Execution rule:
- Run the smallest safe validation immediately after generating files.
- Validate every computed resource name against the naming contract and the target Azure resource type rules before Terraform plan.
- Check Azure name availability for newly generated globally unique names before plan; stop and request a new uniqueness suffix if a name is unavailable.
- Check access before execution: Azure auth, GitHub permissions, required repo variables or secrets, and approval gates.
- If access is missing, prompt the user for access or describe the exact missing prerequisite.
- If Azure or GitHub execution is possible, continue automatically with prerequisite checks, bootstrap steps, script execution, and deployment-safe workflow or Terraform actions.
- After all required inputs and naming checks pass, stop only for missing access, secrets, approvals, or external service blockers.
```