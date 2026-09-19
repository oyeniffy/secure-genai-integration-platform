# ADR-0004: Content Filtering Strategy

## Status
Accepted

## Context
The business problem this repo solves requires content filtering on both the prompt
(input) and completion (output) path — a bank or telco's compliance function will not
sign off on unfiltered model output reaching an end user or downstream system. Azure
AI Content Safety, integrated natively with Azure OpenAI (ADR-0001), provides built-in
filtering categories and jailbreak/prompt-injection detection (Prompt Shields) without
additional infrastructure. The alternative is a custom filtering layer built and
maintained independently of the platform.

## Decision
Use **Azure AI Content Safety's built-in filters** (Hate, Sexual, Violence, Self-Harm
categories, plus Prompt Shields for jailbreak and indirect prompt injection detection)
as the content filtering layer for this pattern. A custom, domain-specific filtering
layer is explicitly out of scope for this version.

## Rationale
- **Native integration, no additional infrastructure.** Content Safety attaches
  directly to the Azure OpenAI resource with no separate service to deploy, network,
  or maintain — consistent with the private-networking boundary established in
  ADR-0002.
- **Covers the generic, well-understood risk surface.** Hate, sexual, violent, and
  self-harm content categories, along with prompt-injection detection, represent the
  risk categories every enterprise compliance review expects to see addressed as a
  baseline, regardless of industry.
- **Configurable severity thresholds.** Content Safety allows per-category severity
  thresholds to be tuned, which gives this pattern room to be adjusted per client
  risk appetite without a code change — only a configuration change.

## Trade-offs and what we're giving up
- **No domain-specific filtering.** Content Safety's categories are generic safety
  categories, not business-context categories. A bank's non-public product names,
  account number formats, or internal terminology appearing in a completion would not
  be caught by any of Content Safety's built-in filters — that is a data-exfiltration
  and confidentiality concern, not a safety concern, and Content Safety is not designed
  to address it.
- **This is a deliberate scope boundary, not an oversight.** A custom filtering layer
  (e.g. regex or classifier-based detection of client-specific sensitive terms) is real
  engineering effort with its own false-positive/false-negative trade-offs, and is
  explicitly deferred rather than attempted partially. The threat model
  (ADR-0006, `docs/threat-model/llm-threat-model.md`) documents this gap under
  LLM02 (Sensitive Information Disclosure) rather than papering over it.
- **Prompt Shields is not a complete prompt-injection defence.** It detects known
  jailbreak and injection patterns but is not a guarantee against novel injection
  techniques — this pattern treats it as one layer of defence, not the only one; see
  ADR-0006 and ADR-0007 for the other layers (identity scoping, tool-calling
  boundaries).

## Consequences
- The Bicep module for Content Safety (`infra/bicep/modules/content-safety.bicep`)
  configures built-in categories and default severity thresholds; no custom
  classifier or filtering microservice is built in this repo.
- The threat model explicitly marks domain-specific data leakage as a documented,
  unmitigated gap rather than a false claim of complete coverage.
- Extending this pattern with a custom filtering layer is noted as a candidate for a
  future portfolio project, not retrofitted here.
