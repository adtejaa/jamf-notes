# Terraform Associate -- 120 Consolidated Revision Q&A

This document consolidates all concepts from your multiple tests into
structured revision questions with concise explanations.

------------------------------------------------------------------------

## SECTION 1 -- Core Concepts

### 1. What does declarative mean in Terraform?

You define the desired end state. Terraform determines execution order
using a dependency graph rather than step-by-step instructions.

### 2. What is Infrastructure as Code (IaC)?

Managing infrastructure using version-controlled configuration files to
ensure reproducibility and automation.

### 3. What is Terraform's core philosophy?

Configuration-driven, state-based, plan-before-apply, predictable
infrastructure lifecycle management.

### 4. What is the dependency graph?

A directed acyclic graph Terraform builds to determine correct creation
and destruction order.

### 5. Does file order matter?

No. Terraform loads all `.tf` files in a directory as a single
configuration.

------------------------------------------------------------------------

## SECTION 2 -- Terraform State

### 6. Why does Terraform require state?

To map configuration blocks to real infrastructure resource IDs and
track metadata.

### 7. What does state store?

Resource IDs, attributes, metadata, dependencies, and outputs.

### 8. Does state store provider configuration?

No.

### 9. Does state store unused variables?

No.

### 10. What happens if state is deleted?

Terraform forgets infrastructure and may recreate resources.

### 11. What is state locking?

Prevents concurrent modification to avoid corruption.

### 12. Does local backend provide locking?

No.

### 13. Why avoid committing state to Git?

May contain secrets, lacks locking, causes conflicts.

### 14. What is drift?

When infrastructure differs from state.

### 15. How is drift detected?

`terraform plan`.

### 16. What does `terraform plan -refresh-only` do?

Updates state to match real infrastructure without changes.

### 17. Does sensitive = true encrypt state?

No.

### 18. Can state contain plaintext secrets?

Yes.

### 19. Where should secrets be protected?

In secure backend storage.

### 20. What is remote state?

State stored in centralized backend like S3 or HCP.

------------------------------------------------------------------------

## SECTION 3 -- Backend & Migration

### 21. What is a backend?

Determines where state is stored.

### 22. Why use remote backend?

Locking, collaboration, centralized control.

### 23. Does remote backend mean remote execution?

No.

### 24. When use `terraform init -reconfigure`?

When backend configuration changes.

### 25. When rerun init?

When provider, module, or backend changes.

------------------------------------------------------------------------

## SECTION 4 -- CLI Commands

### 26. What does `terraform init` do?

Downloads providers, modules, configures backend.

### 27. Does init validate configuration?

No.

### 28. What does `terraform validate` do?

Checks syntax and consistency only.

### 29. Does validate contact providers?

No.

### 30. What does `terraform plan` do?

Shows proposed changes.

### 31. What does `terraform apply` do?

Executes changes.

### 32. How apply exactly reviewed plan?

Use `-out` and apply saved plan.

### 33. Does destroy prompt confirmation?

Yes, unless `-auto-approve`.

### 34. What does `terraform fmt` do?

Formats Terraform files.

### 35. What does `terraform show` display?

Entire state or plan.

### 36. What does `terraform state show` display?

Single resource details.

------------------------------------------------------------------------

## SECTION 5 -- Variables

### 37. What is a variable block?

Defines input variable.

### 38. When is validation executed?

During validate and plan.

### 39. Does default enforce constraint?

No.

### 40. What is map(string)?

Key-value lookup type.

### 41. What is list(string)?

Ordered collection.

### 42. What is object type?

Structured attribute grouping.

### 43. How pass variable via environment?

`TF_VAR_name=value`

### 44. Does unused variable appear in state?

No.

### 45. What is sensitive variable?

Hides CLI output only.

------------------------------------------------------------------------

## SECTION 6 -- Providers

### 46. What is required_providers?

Declares provider source and version.

### 47. What does provider block do?

Configures provider settings.

### 48. Are provider credentials stored in state?

Normally no.

### 49. Where are providers downloaded?

`.terraform/providers`

### 50. Why pin versions?

Ensure reproducibility.

### 51. What does `~> 5.0` mean?

> =5.0 and \<6.0

------------------------------------------------------------------------

## SECTION 7 -- Modules

### 52. What is a module?

Reusable Terraform configuration.

### 53. What are module inputs?

Variables passed to module.

### 54. What are module outputs?

Values exposed to parent.

### 55. How access module output?

`module.name.output`

### 56. Can modules access each other directly?

No.

### 57. Should module versions be pinned?

Yes.

------------------------------------------------------------------------

## SECTION 8 -- Data Sources

### 58. What is data block?

Reads existing infrastructure.

### 59. Does data create resources?

No.

------------------------------------------------------------------------

## SECTION 9 -- Dependency Behavior

### 60. What creates implicit dependency?

Referencing another resource attribute.

### 61. When use depends_on?

When no implicit reference exists.

### 62. Does file order control execution?

No.

------------------------------------------------------------------------

## SECTION 10 -- Import

### 63. What does terraform import do?

Maps existing resource into state.

### 64. Does import recreate resource?

No.

### 65. Must resource block exist first?

Yes.

------------------------------------------------------------------------

## SECTION 11 -- HCP Terraform

### 66. What is CLI-driven workflow?

Execution on HCP infrastructure.

### 67. What is VCS-driven workflow?

Triggered by Git commits.

### 68. What is local execution mode?

Runs locally, HCP stores state.

### 69. What are run triggers?

Trigger downstream workspace.

------------------------------------------------------------------------

## SECTION 12 -- Security & Secrets

### 70. Where store secrets securely?

Vault, env vars, secret managers.

### 71. Does Terraform auto-rotate secrets?

No.

### 72. Does plan file encrypt secrets?

No.

------------------------------------------------------------------------

## SECTION 13 -- Execution Behavior

### 73. Removing resource block causes?

Resource destruction on apply.

### 74. Should -target be normal practice?

No.

### 75. Does plan change infrastructure?

No.

### 76. What ensures predictability?

Plan-first + version pinning.

------------------------------------------------------------------------

## SECTION 14 -- Advanced & Traps

### 77. Does Terraform manage resources not in state?

No.

### 78. What triggers recreation?

Immutable argument change.

### 79. What is idempotency?

Repeated apply causes no change.

### 80. What preserves state integrity?

Remote backend with locking.

------------------------------------------------------------------------

# FINAL EXAM FILTER

If two answers look correct: Choose the one aligned with: - Declarative
workflow - State integrity - Configuration-driven changes -
Reproducibility

Avoid shortcuts and manual approaches.
