# Network, NSG, and Route Table Questionnaire

Use this template to gather required networking inputs before ALZ generation.

## 1) Global Addressing Strategy
- What RFC1918 ranges are approved for this platform?
- What ranges are reserved for future growth?
- What ranges are disallowed due to overlap with on-prem, partner, or acquired networks?
- What is the CIDR allocation strategy per environment (prod, nonprod, shared services)?

Answers:
- Approved ranges:
- Reserved ranges:
- Disallowed/overlap ranges:
- Allocation strategy:

## 2) Region and VNet Address Spaces
- Primary region hub VNet CIDR:
- Secondary region hub VNet CIDR:
- Planned spoke CIDR blocks by app/team:
- CIDR growth buffer policy (minimum free space percentage):

Answers:
- Primary hub CIDR:
- Secondary hub CIDR:
- Spoke CIDR plan:
- Growth buffer:

## 3) Subnet Standards
- Required subnets per spoke (for example: app, data, private-endpoints, integration):
- Minimum subnet sizes by subnet type:
- Delegations required by subnet:
- Private endpoint network policy behavior:

Answers:
- Required subnet set:
- Minimum sizes:
- Delegations:
- Private endpoint policy behavior:

## 4) Default NSG Baseline
- Default inbound deny policy (yes/no and exceptions):
- Default outbound allow/deny policy:
- Required platform allow rules (DNS, NTP, identity, management):
- East-west traffic policy between spokes:
- Rule priority ranges reserved for platform vs app teams:
- Logging/flow log requirements:

Answers:
- Inbound baseline:
- Outbound baseline:
- Platform allow rules:
- East-west policy:
- Priority convention:
- Logging requirements:

## 5) Default Route Table Baseline
- Default route behavior per subnet type (internet, firewall, NVA, virtual appliance):
- Forced tunneling required (yes/no):
- 0.0.0.0/0 next hop target:
- Service tag-specific routes required:
- On-prem route propagation/BGP expectations:
- Blackhole routes required:

Answers:
- Default route behavior:
- Forced tunneling:
- Default next hop:
- Service tag routes:
- BGP expectations:
- Blackhole routes:

## 6) Egress and Inspection
- Central egress pattern (Azure Firewall, NVA, hybrid):
- TLS inspection requirement (yes/no):
- FQDN/application rules required at baseline:
- Exceptions approval process:

Answers:
- Egress pattern:
- TLS inspection:
- Baseline app/FQDN rules:
- Exceptions process:

## 7) Connectivity and Name Resolution
- DNS forwarder/resolver model:
- Private DNS zone linking strategy:
- Cross-region DNS failover expectation:
- Required connectivity to on-prem and partner networks:

Answers:
- DNS model:
- Private DNS linking:
- DNS failover:
- External connectivity requirements:

## 8) Compliance and Guardrails
- Regulatory requirements influencing NSG/route defaults:
- Deny policy requirements for risky ports/protocols:
- Required tags for NSG rules and route tables:

Answers:
- Regulatory controls:
- Deny controls:
- Tagging standard:

## 9) Change and Ownership
- Owner of default NSG baseline:
- Owner of default route table baseline:
- SLA for emergency rule changes:
- Break-glass process:

Answers:
- NSG owner:
- Route table owner:
- Emergency SLA:
- Break-glass process:

## 10) Final Generation-Ready Inputs
- Confirm the exact default NSG rules to generate:
- Confirm the exact default route entries to generate:
- Confirm IP CIDR sets to use in tfvars:

Answers:
- NSG rules to generate:
- Route entries to generate:
- CIDR sets for tfvars:
