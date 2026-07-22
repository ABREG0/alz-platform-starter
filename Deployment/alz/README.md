# Azure Landing Zone Deployment Artifacts

This folder contains the repo-scoped artifacts needed to create and deploy an Azure Landing Zone environment from the guidance in [../1-file-to-give-to-AI.md](../1-file-to-give-to-AI.md).

For this prompt test, every generated artifact must stay inside `Deployment/alz`. Do not promote, copy, or adapt outputs into active repo roots.

## Purpose

Use these files to:

- collect the environment-specific values that the source markdown requires
- collect one strict resource naming contract before any names are generated
- generate ALZ environment artifacts into this folder instead of scattering them across the repo
- execute the existing repo deployment path for the ALZ full multi-region stack

## High-level design map

Open the interactive design document to visualize what the prompts generate and how the deployment path flows across inputs, generation, validation, deployment, and governance gates:

- [alz-prompt-deployment-route-explorer.html](./alz-prompt-deployment-route-explorer.html)

## Contents

- [platform-spec.template.md](./platform-spec.template.md): fill-in template for the required environment values
- [generate-alz-environment.prompt.md](./generate-alz-environment.prompt.md): deterministic prompt that tells an AI to generate all requested artifacts under `./Deployment/alz/generated`
- [deploy-runbook.md](./deploy-runbook.md): repo-specific steps to validate and deploy after generation
- [network-security-routing-questionnaire.template.md](./network-security-routing-questionnaire.template.md): intake template for IP spaces, default NSG rules, and route table defaults
- [prompts/AI-agent-handoff-map.md](./prompts/AI-agent-handoff-map.md): exact file-selection guide for what to pass to AI or coding agents

## ALZ scope guardrails (required)

All ALZ prompt loops in this folder must include:

- Management group hierarchy deployment.
- Role assignments at management group scope.
- ALZ policy deployment plus AMBA onboarding/configuration.
- No subscription moves into management groups unless explicitly requested.
- CAF-aligned naming derived from the PLATFORM SPEC, with Azure constraints enforced and exact exceptions required for existing names.

## Current repo implementation anchor

This artifact pack is aligned to the existing working stack at:

- [../../terraform/environments/dev/alz-full-multi-region/main.tf](../../terraform/environments/dev/alz-full-multi-region/main.tf)
- [../../terraform/environments/dev/alz-full-multi-region/run-platform-alz-prod.ps1](../../terraform/environments/dev/alz-full-multi-region/run-platform-alz-prod.ps1)
- [../../terraform/environments/dev/alz-full-multi-region/_docs_scripts/alz-full-multi-region-repro-prompts.md](../../terraform/environments/dev/alz-full-multi-region/_docs_scripts/alz-full-multi-region-repro-prompts.md)

## Important constraint

Actual Azure deployment cannot be executed safely until `platform-spec.template.md` is filled with real tenant, subscription, backend, runner, RBAC, and naming values, and the operator is authenticated to Azure.

## Expected output location

Generated files should be written under:

- `./Deployment/alz/generated/.github/workflows`
- `./Deployment/alz/generated/scripts`
- `./Deployment/alz/generated/terraform`

Keep all generated artifacts in this folder. For this test, there is no promotion step into active repo roots.
