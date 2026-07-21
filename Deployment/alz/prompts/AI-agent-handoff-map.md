# ALZ Prompt and Loop Handoff Map

Use this file to select exactly which prompt file to pass to an AI or coding agent.

## Primary Files to Pass

1. Full ALZ generate-to-deploy operator loop
- File: ../prompts/alz-merged-operator.prompt.md
- Pass when: you want one operator prompt to generate, stage, review, preflight, validate, deploy, and triage.

2. Tight implementation/debug loops
- File: ../prompts/loop-engineering-agent.prompt.md
- Pass when: you already have a concrete failing file, command, workflow step, or Terraform error.

3. Deterministic ALZ file generation
- File: ../generate-alz-environment.prompt.md
- Pass when: you need complete generated files under Deployment/alz/generated from PLATFORM SPEC.

4. Required values source of truth
- File: ../platform-spec.template.md
- Pass when: gathering/confirming tenant, subscriptions, MG hierarchy, RBAC scope, policy/AMBA, backend, and runners.

5. Network intake before generation
- File: ../network-security-routing-questionnaire.template.md
- Pass when: you need IP space, default NSG, and route table baseline decisions finalized.

## Required ALZ Scope Checks

Every ALZ run must include all of these unless user explicitly narrows scope:

1. Management group hierarchy deployment.
2. Role assignments at management group scope.
3. ALZ policy deployment plus AMBA.
4. No subscription moves into management groups unless explicitly requested.
5. Resource names derived from the naming contract in PLATFORM SPEC.
6. Azure resource-specific naming rules validated before generation or plan.
7. No naming exception without an exact name and reason in PLATFORM SPEC.

## Suggested Run Order

1. Fill ../network-security-routing-questionnaire.template.md.
2. Fill ../platform-spec.template.md with finalized platform and naming values.
3. Validate the naming contract and resolve or approve any exact existing-name exceptions.
4. Run ../generate-alz-environment.prompt.md for deterministic artifact output.
5. Run ../prompts/alz-merged-operator.prompt.md for preflight/validate/deploy loops.
6. If a specific failure appears, switch to ../prompts/loop-engineering-agent.prompt.md until resolved.
