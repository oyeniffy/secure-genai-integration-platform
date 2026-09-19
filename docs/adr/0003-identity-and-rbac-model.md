# ADR-0003: Identity and RBAC Model

## Status
Accepted

## Context
ADR-0001 and ADR-0002 establish Azure OpenAI behind a private endpoint. Whatever
service calls that endpoint (an API, a Function App, an orchestration layer) needs an
identity and a permission model to do so. Azure OpenAI supports both API-key and
Microsoft Entra ID (managed identity) authentication. This repo's business context —
an enterprise client requiring full audit logging and no standing credentials — makes
the identity choice a compliance decision, not just a convenience one.

## Decision
Use a **user-assigned managed identity**, granted a **custom RBAC role** scoped to
inference-only operations on the Azure OpenAI resource, following the same
identity-and-RBAC pattern established in
[multicloud-zero-trust-landing-zone](https://github.com/oyeniffy/multicloud-zero-trust-landing-zone).

## Rationale
- **No standing credentials.** API keys are long-lived secrets that must be stored,
  rotated, and can leak. A managed identity removes the credential entirely — access is
  granted through Entra ID token issuance, which is auditable and revocable at the
  identity level.
- **User-assigned over system-assigned.** A system-assigned identity is tied to the
  lifecycle of a single resource and cannot be provisioned or referenced independently.
  A user-assigned identity can be created ahead of the consuming resource, referenced by
  name in the Bicep modules, and reused if the consuming service is replaced —
  consistent with how identity is treated as a first-class, independently managed
  resource in the landing zone repo.
- **Custom role over the built-in role.** Azure's built-in `Cognitive Services OpenAI
  User` role grants inference access but does not distinguish between read-only
  completion calls and any future administrative or configuration action on the
  resource. A custom role definition, scoped explicitly to the inference data-plane
  action, is a more defensible least-privilege claim for a compliance audience than
  relying on a built-in role's broader intent.

## Trade-offs and what we're giving up
- **Custom role maintenance.** A custom RBAC role definition must be maintained
  independently of Microsoft's built-in role updates — if Azure OpenAI's action
  namespace changes, the custom role's action list needs to be revisited. The built-in
  role would not require this.
- **Slightly higher setup complexity.** A user-assigned identity requires an explicit
  resource, explicit role assignment, and explicit reference from the consuming
  service — more moving parts than a system-assigned identity's implicit lifecycle
  binding, in exchange for the reusability and auditability above.

## Consequences
- The Bicep module for this identity (`infra/bicep/modules/managed-identity-rbac.bicep`)
  defines the custom role and its assignment explicitly, rather than referencing a
  built-in role ID.
- Audit logging (ADR-0005) can attribute every prompt/response event to this specific
  identity, which is a precondition for the audit trail the business problem requires.
- Any future consuming service for this Azure OpenAI resource is expected to be granted
  the same custom role rather than a new bespoke permission set, keeping the RBAC model
  consistent as the pattern is reused.
