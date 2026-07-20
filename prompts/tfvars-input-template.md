Collect and validate deployment inputs, then generate final `terraform.tfvars` and `backend.hcl` content.

## Inputs (required)

### Global

- `deployment_type`: `connectivity` | `vwan-connectivity` | `app-landing-zone` | `selfhosted-aks` | `vnet-integrated-runners`
- `environment_name`: `dev` | `prod`
- `name_suffix` (example: `dev001`)
- `primary_region` (default: `eastus2`)
- `secondary_region` (default: `centralus`)
- `caf_alignment_profile` (`foundational` | `enterprise` | `regulated`)
- `waf_prioritized_pillars` (ordered list from `reliability`, `security`, `cost_optimization`, `operational_excellence`, `performance_efficiency`)
- `subscription_vending_strategy` (`none` | `msft-vending-workflows` | `avm-subscription-module` | `hybrid`)
- `subscription_lifecycle_owner` (`platform-team` | `app-team` | `central-it`)
- `subscription_move_policy` (`create-only` | `create-and-move`)
- `avm_subscription_module_version` (required when `subscription_vending_strategy` is `avm-subscription-module` or `hybrid`)
- `target_management_group_id` (required when vending strategy is not `none`)

### Subscription mapping

- `connectivity_subscription_id`
- `identity_subscription_id`
- `security_management_subscription_id`

### Connectivity-only

- `dns_resource_group_name`
- `primary_hub_address_space` (list)
- `primary_hub_routing_address_space` (list)
- `secondary_hub_address_space` (list)
- `secondary_hub_routing_address_space` (list)
- `primary_dns_resolver_subnet_cidr`
- `secondary_dns_resolver_subnet_cidr`
- `primary_dns_resolver_forwarding_domain` (FQDN, trailing dot)
- `secondary_dns_resolver_forwarding_domain` (FQDN, trailing dot)
- `primary_dns_forwarder_ips` (list)
- `secondary_dns_forwarder_ips` (list)
- `monitoring_resource_group_name`
- `log_analytics_workspace_name`
- `log_analytics_retention_in_days` (default: `30`)

### vWAN-only

- `vwan_resource_group_name`
- `primary_hub_address_prefix` (CIDR, recommended min `/23`)
- `secondary_hub_address_prefix` (CIDR, recommended min `/23`)
- `primary_express_route_circuit_peering_id` (must end with `/peerings/AzurePrivatePeering`)
- `secondary_express_route_circuit_peering_id` (must end with `/peerings/AzurePrivatePeering`)
- `virtual_wan_type` (recommended: `Standard`)
- `allow_branch_to_branch_traffic` (recommended: `true`)
- `virtual_router_auto_scale_min_capacity` (recommended: `2`)
- `express_route_gateway_scale_units` (recommended: `2`)
- `express_route_routing_weight` (recommended: `100`)

### App landing zone only

- `app_subscription_id`
- `legacy_state_resource_group_name`
- `legacy_state_storage_account_name`
- `legacy_state_container_name`
- `legacy_state_key`
- `application_name`
- `application_abbreviation`
- `app_vnet_address_space`
- `vm_subnet_address_prefixes`
- `private_endpoint_subnet_address_prefixes`
- `web_app_name`
- `app_service_plan_name`
- `app_service_plan_sku`
- `storage_account_name` (global unique, lowercase, 3-24)
- `key_vault_name` (global unique)
- `web_app_private_dns_zone_name` (default: `privatelink.azurewebsites.net`)
- `storage_blob_private_dns_zone_name` (default: `privatelink.blob.core.windows.net`)
- `key_vault_private_dns_zone_name` (default: `privatelink.vaultcore.azure.net`)
- `linux_vm_name`
- `linux_admin_username`
- `linux_admin_ssh_public_key`
- `windows_vm_name`
- `windows_admin_username`
- `windows_admin_password`

### Runner roots only

- `app_subscription_id`
- `legacy_state_resource_group_name`
- `legacy_state_storage_account_name`
- `legacy_state_container_name`
- `legacy_state_key`

### Tags

- `owner`
- `managed_by` (default: `terraform`)
- `workload`
- `environment`
- `cost_center` (optional)
- `business_unit` (optional)

### Backend

- `resource_group_name`
- `storage_account_name`
- `container_name`
- `key`
- `use_oidc = true`
- `use_azuread_auth = true`

## Default Recommended Profile

Use this profile only when the user does not provide explicit values and policy allows defaults:

- `caf_alignment_profile`: `enterprise`
- `waf_prioritized_pillars`: `security`, `reliability`, `operational_excellence`, `cost_optimization`, `performance_efficiency`
- `subscription_vending_strategy`: `avm-subscription-module`
- `subscription_lifecycle_owner`: `platform-team`
- `subscription_move_policy`: `create-only`
- `target_management_group_id`: required (do not invent)
- `avm_subscription_module_version`: required (do not invent)

## Validation Rules (must enforce)

1. Primary/secondary regions must differ.
2. Hub CIDRs must not overlap.
3. Any ExpressRoute peering ID must end with `/peerings/AzurePrivatePeering`.
4. No placeholder values (`<...>`, `REPLACE_ME`, `TBD`, `00000000-0000-0000-0000-000000000000`).
5. Backend `key` must match deployment root and environment.
6. For app landing zone, workload naming must use `application_abbreviation`.
7. `waf_prioritized_pillars` must include all five WAF pillars exactly once.
8. If `subscription_vending_strategy` is `avm-subscription-module` or `hybrid`, `avm_subscription_module_version` is required.
9. If `subscription_vending_strategy` is not `none`, `target_management_group_id` is required.
10. If `subscription_move_policy` is `create-only`, do not emit steps that move existing subscriptions between management groups.

## Output Contract (exact order)

1. `TERRAFORM_TFVARS` (complete block)
2. `BACKEND_HCL` (complete block)
3. `MISSING_OR_INVALID_VALUES` (checklist; write `none` if empty)
4. `RULES_CHECK` (pass/fail for each rule)
