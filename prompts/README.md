# Prompt Library

This folder contains production-ready prompts for planning, deploying, validating, and troubleshooting this ALZ repository.

## Prompt Quality Standard

Every prompt in this folder follows these optimization rules:

1. Put **Inputs first** (all required values listed up front).
2. Use **explicit gates** (`stop if ...`) to prevent unsafe partial output.
3. Force **deterministic output** (fixed sections, fixed order).
4. Keep prompts **task-scoped** (implementation, input collection, troubleshooting, handoff).
5. Keep values parameterized; avoid hard-coded environment-specific assumptions.

## Repo Baseline Assumptions

- Primary roots:
  - `terraform/environments/<env>/connectivity`
  - `terraform/environments/<env>/vwan-connectivity`
  - `github_runners/vnet_integrated`
  - `github_runners/selfhosted_aks`
- Environments: `dev`, `prod`
- Backend auth: AzureRM with OIDC + Azure AD auth
- Active input files are real files (`terraform.tfvars`, `backend.hcl`) and not `.example` templates
- Operational validation scripts:
  - `scripts/smoke-check.ps1`
  - `scripts/validate-deployment.ps1`
- ALZ workflow dispatch safety:
  - advanced workflow dispatchers require explicit `action` and support a destroy approval gate via `destroy_approved`
  - use `destroy_approved=yes` only for intentional destroy runs

## Prompt Index

- `deployment-assistant.md`: implementation/change execution prompt with strict gates
- `tfvars-input-template.md`: input intake and generation template for `terraform.tfvars` + `backend.hcl`
- `workflow-ops-template.md`: CI/CD OIDC/backend/runtime troubleshooting prompt
- `new-team-app-onboarding.md`: app team onboarding and readiness assessment prompt
- `new-team-app-deploy.md`: app team deployment execution prompt
- `new-team-app-validation.md`: post-deployment validation and handoff prompt
- `session-handoff-2026-02-22.md`: reusable continuity prompt containing latest decisions, constraints, and open checks
- `work-completed-2026-02-22.md`: saved structured summary prompt of work completed so far
- `master-operator-prompt.md`: unified operator prompt for deploy/validate/troubleshoot modes
- `master-operator-prompt-specit.md`: SPEC-IT formatted copy of the unified operator prompt
- `subscription-vending-artifact-export.md`: generate all vending artifacts (workflow YML, optfile JSON, setup script)
- `subscription-vending-workflow-yml-generator.md`: generate subscription vending workflow dispatcher YAML
- `subscription-vending-optfile-json-generator.md`: generate subscription vending optfile JSON inputs
- `subscription-vending-rbac-script-generator.md`: generate OIDC federated credential + RBAC setup PowerShell script
- `Deployment/alz/prompts/AI-agent-handoff-map.md`: exact file map for which ALZ prompt/loop file to pass to AI or agent
- `Deployment/alz/network-security-routing-questionnaire.template.md`: IP space, default NSG, and route table questionnaire template
- `terraform/environments/dev/alz-full-multi-region/_docs_scripts/alz-full-multi-region-repro-prompts.md`: prompt definitions to reproduce the exact full multi-region workflow/local lifecycle
- `terraform/environments/dev/alz-full-multi-region/_docs_scripts/constitution.md`: Spec-Kit constitution for SDD governance and execution gates
- `terraform/environments/dev/alz-full-multi-region/flows_n_optfile/`: portable package of working workflow and optfile templates for repo/tenant/subscription migration

## Selection Matrix

- Implement infra changes: `deployment-assistant.md`
- Collect deployment values: `tfvars-input-template.md`
- Diagnose workflow/runtime failures: `workflow-ops-template.md`
- Onboard new app team: `new-team-app-onboarding.md`
- Execute app rollout: `new-team-app-deploy.md`
- Perform go-live validation: `new-team-app-validation.md`
- Continue from current state: `session-handoff-2026-02-22.md`
- Review completed execution context: `work-completed-2026-02-22.md`
- Run one prompt for all operations: `master-operator-prompt.md`
- Use spec-style prompt format: `master-operator-prompt-specit.md`
- Generate all subscription vending inputs/artifacts: `subscription-vending-artifact-export.md`
- Generate subscription vending workflow YAML only: `subscription-vending-workflow-yml-generator.md`
- Generate subscription vending optfile JSON only: `subscription-vending-optfile-json-generator.md`
- Generate subscription vending setup script only: `subscription-vending-rbac-script-generator.md`
- Use ALZ prompt/loop file handoff map: `Deployment/alz/prompts/AI-agent-handoff-map.md`
- Collect network, NSG, and route baseline inputs: `Deployment/alz/network-security-routing-questionnaire.template.md`
- Reproduce full multi-region deployment flow: `terraform/environments/dev/alz-full-multi-region/_docs_scripts/alz-full-multi-region-repro-prompts.md`
- Run Spec-Kit SDD with repo constitution: `terraform/environments/dev/alz-full-multi-region/_docs_scripts/constitution.md`
- Reuse working workflow/optfile templates in another repo: `terraform/environments/dev/alz-full-multi-region/flows_n_optfile/`

## Maintenance Protocol

Update prompt files in the same PR whenever code, workflows, or runbooks change:

1. Update required fields/defaults in `tfvars-input-template.md`.
2. Update execution and validation steps in `deployment-assistant.md`.
3. Add new failure patterns/checks in `workflow-ops-template.md`.
4. Keep output contracts stable unless intentionally versioned.
5. Always optimize prompt + Spec-Kit artifacts from latest execution evidence (new guardrails, defaults, and failure signatures).
6. Keep workflow-dispatch inputs aligned with current reusable workflow contracts (`action`, `destroy_approved`, `optfile_path`, `environment_name`, `azure_client_id`, `azure_tenant_id`, `ref`).

## Usage

Copy one prompt, fill the inputs, and run it as-is. If you modify a prompt format, update this README selection matrix and maintenance notes.
