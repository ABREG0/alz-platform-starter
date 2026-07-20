You are troubleshooting GitHub Actions OIDC Terraform workflow failures and backend/runtime issues in this repository.

## Inputs (required)

- `failing_workflow_name`
- `run_url_or_id`
- `failing_job_name`
- `deployment_type` (`connectivity` | `vwan-connectivity` | `app-landing-zone` | `selfhosted-aks` | `vnet-integrated-runners`)
- `environment` (`dev` | `prod`)
- `error_snippet` (first 30-80 relevant log lines)

## Triage Sequence (run in order)

1. **Workflow permissions**
	- Confirm `permissions: id-token: write, contents: read`.
2. **Azure login wiring**
	- Confirm `azure/login` uses `AZURE_CLIENT_ID`, `AZURE_TENANT_ID`, `AZURE_SUBSCRIPTION_ID`.
3. **OIDC federation subject**
	- Confirm Entra federated credential subject exactly matches branch/environment claim used by workflow.
4. **Backend correctness**
	- Validate `resource_group_name`, `storage_account_name`, `container_name`, `key`, `use_oidc`, `use_azuread_auth`.
5. **Identity authorization**
	- Management plane: `Contributor` at deploy scope.
	- Data plane: `Storage Blob Data Contributor` (or higher) for tfstate scope.
6. **Backend network reachability**
	- Verify storage firewall/private endpoint model allows runner path.
	- Explicitly check `publicNetworkAccess`; if `Disabled`, require private network path (or hosted runner path) to backend storage.
7. **Environment protections**
	- Confirm approvals and protection rules are satisfied.
8. **Deployment-type checks**
	- `vwan-connectivity`: peering IDs valid + `AzurePrivatePeering` + non-overlapping hub CIDRs.
	- `app-landing-zone`: bidirectional peering permissions + central DNS write permissions + legacy state outputs present.
	- `selfhosted-aks`: cluster identity can write private DNS zone and has subnet permissions.
	- `connectivity`: DNS RG naming uses short region codes (for example `eus2`, `cus`) and DNS zones are linked to expected VNets.

## High-Probability Failure Patterns

- `403 AuthorizationFailure` during `terraform init`/lock/read state.
- `publicNetworkAccess=Disabled` on backend storage account while running from a non-private local path.
- OIDC token minted but federated credential subject mismatch.
- Invalid or placeholder subscription IDs.
- Backend drift between local and workflow-generated config.
- ExpressRoute peering ID malformed or unauthorized.
- Private DNS link/record writes blocked by missing role assignment.
- Peering fails because identity has one-sided permissions.
- DNS zones created in unexpected RG due to stale short-code naming in tfvars.
- DNS zones present but no VNet links for one or more zones.

## Stop Conditions

- If logs are missing, request exact failing job log segment before continuing.
- If multiple independent root causes are found, rank by blast radius and remediation speed.

## Output Contract (exact order)

1. `PRIMARY_ROOT_CAUSE`
2. `CONTRIBUTING_FACTORS`
3. `FIX_PLAN_ORDERED`
4. `EXACT_FILE_OR_CONFIG_CHANGES`
5. `VERIFICATION_COMMANDS`
6. `PREVENTION_GUARDRAIL`

## Azure Verification Commands (preferred)

- `az network private-dns zone list -g <dns-rg> -o table`
- `az network private-dns zone list -g <dns-rg> -o json | ConvertFrom-Json | Where-Object { $_.numberOfVirtualNetworkLinks -eq 0 }`
- `az storage account show --resource-group <rg> --name <storage-account> --query "{publicNetworkAccess:publicNetworkAccess,defaultAction:networkRuleSet.defaultAction}" -o json`
- `./scripts/grant-backend-blob-rbac.ps1 -SubscriptionId <subId> -ResourceGroupName <rg> -StorageAccountName <storage-account> -ContainerName tfstate` (append `-Apply` to execute)
