# ADR-0008: Data Residency Constraint

## Status
Accepted

## Context
"Data residency" is not a single, universal requirement — it varies by the client's
jurisdiction and the specific regulation that applies to them. A pattern that claims to
solve data residency in the abstract is making a claim it cannot actually support. This
repo instead needs to demonstrate the *mechanism* for enforcing residency, using
concrete regulatory examples relevant to the markets this portfolio targets: Nigeria
(NDPR — Nigeria Data Protection Regulation) and the United Kingdom (UK GDPR).

## Decision
Pin the Azure OpenAI deployment to a **specific Azure region matching the client's
regulatory jurisdiction**, enforced via Azure Policy (deny-by-default outside the
approved region), rather than leaving region selection as an unconstrained deployment
parameter. This repo documents two worked examples — a Nigerian client under NDPR and
a UK client under UK GDPR — rather than a single hardcoded region.

## Rationale
- **Region pinning is enforceable; a documentation promise is not.** Stating in a
  README that "data stays in-region" is not a control. An Azure Policy that denies
  deployment of the Cognitive Services resource outside an approved region list is an
  enforced technical control, consistent with the policy-as-code approach in
  [azure-cloud-governance-finops-platform](https://github.com/oyeniffy/azure-cloud-governance-finops-platform).
- **Two worked examples demonstrate the pattern generalizes, without overclaiming.**
  NDPR and UK GDPR have different specifics (NDPR's registration and data protection
  officer requirements differ from UK GDPR's), but the enforcement mechanism —
  policy-as-code region pinning — is identical. Showing both proves the pattern is
  reusable, not that it satisfies every regulation by default.
- **This connects data residency to the private networking decision.** ADR-0002's
  private endpoint keeps traffic off the public internet; this ADR keeps the data
  itself pinned to a specific region. The two controls address different parts of the
  same compliance conversation and are documented as complementary, not redundant.

## Trade-offs and what we're giving up
- **This is not legal advice or regulatory certification.** Pinning a region is a
  necessary technical control for data residency compliance, not a sufficient one — an
  actual compliance sign-off requires legal review specific to the client's full data
  processing activity, which is out of scope for an infrastructure pattern.
- **Region availability constrains model access.** Not every Azure region has Azure
  OpenAI model availability, and available models/versions can differ by region — a
  client's residency requirement may force a trade-off against model capability that
  this ADR flags but does not resolve generically, since it depends on the specific
  region and model in question at deployment time.
- **NDPR and UK GDPR are the two examples chosen, not an exhaustive list.** A client
  under a different regulatory regime (e.g. EU GDPR, POPIA) would need the same
  mechanism applied to their specific jurisdiction — the pattern generalizes, the two
  examples in this repo do not.

## Consequences
- Azure Policy definitions (`infra/policy/`) include a region-restriction policy
  parameterized by an approved region list, not a single hardcoded value.
- The architecture overview documents both worked examples (NDPR, UK GDPR) side by
  side, showing the same infrastructure pattern with different region parameters.
- Any deployment of this pattern for a specific client requires that client's
  applicable regulation to be identified explicitly before the region parameter is
  set — this is called out as a required input, not a default.
