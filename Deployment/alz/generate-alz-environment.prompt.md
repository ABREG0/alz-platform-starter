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

This is a prompt test. Write every generated file under `./Deployment/alz/generated` and preserve this structure exactly.
Do not create, update, or suggest promotion into `.github`, `scripts`, `terraform`, `optfiles`, or any other root outside `Deployment/alz`.
After generation, continue automatically with validation and the next safe execution step inside `Deployment/alz` whenever the environment allows it.
Before any execution step, check whether Azure access, GitHub access, required secrets, and approval paths are available. If not, ask the user for the missing access or describe the exact configuration needed.

Generate:

1. `./Deployment/alz/generated/.github/workflows/Bootstrap.yaml`
   - workflow_dispatch trigger, no inputs
   - OIDC login using azure/login@v3.0.0 with vars.AZURE_CLIENT_ID / AZURE_TENANT_ID
   - creates the state storage resource group and storage account
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
   - `spoke_components` schema supporting `network`, `nsg`, and `route-table`

10. `./Deployment/alz/generated/terraform/spoke-unified/prod/main.tf`
    - filters enabled spoke components by component type
    - calls modules for network, NSG, and route table components

11. `./Deployment/alz/generated/terraform/spoke-unified/prod/modules/spoke-network/main.tf`
12. `./Deployment/alz/generated/terraform/spoke-unified/prod/modules/spoke-network/variables.tf`
13. `./Deployment/alz/generated/terraform/spoke-unified/prod/modules/spoke-nsg/main.tf`
14. `./Deployment/alz/generated/terraform/spoke-unified/prod/modules/spoke-nsg/variables.tf`
15. `./Deployment/alz/generated/terraform/spoke-unified/prod/modules/spoke-route-table/main.tf`
16. `./Deployment/alz/generated/terraform/spoke-unified/prod/modules/spoke-route-table/variables.tf`
17. `./Deployment/alz/generated/terraform/spoke-unified/prod/tfvars/prod.tfvars`
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

After generating the files, output these sections in order:
1. GENERATED_FILES
2. ASSUMPTIONS
3. AUTOMATIC_ACTIONS
4. VALIDATION_COMMANDS
5. STAGING_NOTES

Execution rule:
- Run the smallest safe validation immediately after generating files.
- Check access before execution: Azure auth, GitHub permissions, required repo variables or secrets, and approval gates.
- If access is missing, prompt the user for access or describe the exact missing prerequisite.
- If Azure or GitHub execution is possible, continue automatically with prerequisite checks, bootstrap steps, script execution, and deployment-safe workflow or Terraform actions.
- Stop only for missing access, secrets, approvals, or external service blockers.
```