# ADR-0001: Platform Choice — Azure OpenAI vs AWS Bedrock

## Status
Accepted

## Context
This platform integrates a large language model into an enterprise workflow for a
regulated client (bank, telco). The two credible managed LLM platforms available at
enterprise scale are Azure OpenAI Service and AWS Bedrock. The choice affects identity,
networking, policy, and observability patterns for every other decision in this repo.

This decision does not happen in isolation: this repo is one of a portfolio of projects
(see [multicloud-zero-trust-landing-zone](https://github.com/oyeniffy/multicloud-zero-trust-landing-zone)
and [azure-cloud-governance-finops-platform](https://github.com/oyeniffy/azure-cloud-governance-finops-platform))
that establish a reusable identity, policy, and governance substrate on Azure.

## Decision
Use **Azure OpenAI Service**, deployed behind a private endpoint, as the LLM platform
for this pattern.

## Rationale
- **Control-plane coherence.** The managed identity, RBAC, and policy-as-code patterns
  established in the landing zone repo apply directly — no second identity model to
  design or explain.
- **Governance reuse.** Budget guardrails and cost anomaly detection from the FinOps
  platform repo extend naturally to Azure OpenAI token consumption, without a second
  cost-control mechanism for a separate cloud.
- **Target market alignment.** UK enterprise clients in financial services and telecoms
  — the audience this portfolio is built for — skew heavily toward Microsoft-centric
  cloud estates. A pattern proven on Azure is more directly transferable to that
  audience's existing environment.
- **Regulatory posture.** Azure OpenAI's data handling commitments (no prompt/completion
  data used for model training, regional data residency options) map cleanly onto the
  compliance concerns this pattern exists to solve.

## Trade-offs and what we're giving up
- **Model flexibility.** Bedrock's multi-model marketplace (Anthropic, Cohere, Meta,
  Amazon's own models) offers more choice and easier model-switching without
  re-architecting the integration layer. Azure OpenAI ties this pattern to OpenAI's
  model family.
- **Guardrails maturity.** Bedrock Guardrails is a purpose-built, model-agnostic content
  and topic filtering layer. Azure AI Content Safety is a comparable capability but is
  architected as an adjacent service rather than an inline guardrail — this shapes the
  design of ADR-0004.
- **Multi-cloud posture.** The landing zone repo demonstrates multi-cloud competency
  (Bicep + Terraform) deliberately. This repo does not extend that multi-cloud story —
  it is intentionally Azure-only, because the reuse argument above outweighs the
  multi-cloud demonstration value for this specific pattern.

## Consequences
- Identity and RBAC design (ADR-0003) reuses the landing zone repo's managed identity
  pattern directly.
- Content filtering (ADR-0004) is designed against Azure AI Content Safety's
  capabilities and gaps, not Bedrock Guardrails.
- A future portfolio project could revisit this decision explicitly to demonstrate the
  Bedrock path and multi-model guardrail design — noted as related but out of scope here.
