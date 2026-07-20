# PLATFORM SPEC

Fill in this file before generating or deploying an Azure Landing Zone environment.

Do not leave placeholder values for any field that identifies Azure tenant, subscriptions, backend resources, GitHub runner placement, or approval boundaries.

```text
PLATFORM SPEC
=============

## Identity
Entra tenant ID: <tenant-guid>
Service principal display name: <service-principal-display-name>
Service principal client ID: <service-principal-client-id>

## Subscriptions
Management subscription ID: <management-subscription-guid>
Connectivity subscription ID: <connectivity-subscription-guid>
Identity subscription ID: <identity-subscription-guid>
Security subscription ID: <security-subscription-guid>
Spoke subscription IDs: <comma-separated-spoke-subscription-guids>

## ALZ management group hierarchy
Root management group ID: <root-mg-id>
Platform management group ID: <platform-mg-id>
Landing zones management group ID: <landing-zones-mg-id>
Corp management group ID: <corp-mg-id>
Online management group ID: <online-mg-id>
Sandbox management group ID: <sandbox-mg-id>
Decommissioned management group ID: <decommissioned-mg-id>

## Subscription placement behavior
Move subscriptions into management group hierarchy: false

## Management group scope RBAC
Principal/object IDs for MG scope RBAC: <comma-separated-object-ids>
Role assignments at MG scope:
- Reader
- Policy Contributor
- Log Analytics Contributor

## ALZ policies and AMBA
Deploy ALZ policies: true
Deploy AMBA baseline: true
AMBA workspace subscription ID: <amba-workspace-subscription-guid>
AMBA workspace resource group: <amba-workspace-rg>
AMBA workspace name: <amba-workspace-name>

## State backend
Storage account name: <storage-account-name>
Resource group name: <backend-resource-group-name>
subscription ID for state storage: <backend-subscription-guid>
Region: <backend-region>
Blob container for state: tfstate
Blob container for plans: tfplan

## GitHub
Organization / Enterprise name: <github-org-or-enterprise>
Repo name(s): <comma-separated-repo-names>
Runner group name: <runner-group-name>
Runner label: <runner-label>
GitHub Environment for plan: <environment-name>
GitHub Environment for apply: <environment-name>-Apply
Apply approver team: <github-team-name>

## Terraform
Terraform version: ~> 1.12
AzureRM provider version: ~> 4.21

## Regions
Primary region: <primary-region>
Secondary region: <secondary-region>

## RBAC roles to assign on each spoke subscription
- Contributor
- Storage Blob Data Contributor

## RBAC roles to assign on state storage resource group (bootstrap only)
- Storage Blob Data Owner
- Storage Account Contributor

## Network, NSG, and route table intake
Use [network-security-routing-questionnaire.template.md](./network-security-routing-questionnaire.template.md) and copy all finalized answers into this PLATFORM SPEC before generation.
```

## Repo mapping notes

- `Management`, `Connectivity`, `Identity`, and `Security` map to the `subscription_ids` object used by [../../terraform/environments/dev/alz-full-multi-region/variables.tf](../../terraform/environments/dev/alz-full-multi-region/variables.tf).
- `Primary region` and `Secondary region` map to `starter_locations` in the ALZ full multi-region stack.
- Backend values must align with the remote state settings used by the deployment workflow or local wrapper script.
