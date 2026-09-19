# LLM Threat Model — Secure GenAI Integration Platform

## Methodology
This threat model uses the OWASP Top 10 for LLM Applications as its baseline taxonomy
(see [ADR-0006](../adr/0006-threat-model-methodology.md)). Each category is mapped
against this specific architecture: relevant categories reference the ADR that
provides the mitigation; categories judged low-relevance or non-applicable are marked
explicitly, with a stated reason, rather than omitted.

## Scope
This threat model covers the architecture defined by ADR-0001 through ADR-0008: a
privately-networked Azure OpenAI deployment, accessed via managed identity, with
content filtering and audit logging. It does not cover application-layer code that
consumes this pattern (a specific client application's business logic is out of
scope), and it does not cover fine-tuning or retrieval-augmented generation, since
neither is part of this pattern's baseline scope.

## Threat mapping

| # | OWASP Category | Relevance | Mitigation |
|---|---|---|---|
| LLM01 | Prompt Injection | High | Azure AI Content Safety Prompt Shields detect known jailbreak and injection patterns ([ADR-0004](../adr/0004-content-filtering-strategy.md)); treated as one defence layer, not a complete guarantee against novel injection techniques |
| LLM02 | Sensitive Information Disclosure | High | Structured metadata logging by default with full content gated behind a restricted break-glass workspace ([ADR-0005](../adr/0005-audit-logging-and-data-retention.md)); domain-specific data leakage (e.g. a client's internal terminology) is a documented, unmitigated gap — Content Safety's categories are generic, not business-context-aware ([ADR-0004](../adr/0004-content-filtering-strategy.md)) |
| LLM03 | Supply Chain | Low | Model versions are pinned at the Azure OpenAI deployment level rather than dynamically resolved; no third-party plugins or extensions are integrated in this pattern's baseline scope |
| LLM04 | Data and Model Poisoning | Not applicable | This pattern does not fine-tune models — it uses Azure OpenAI's managed models as-is. Poisoning risk applies to training data pipelines, which do not exist in this architecture |
| LLM05 | Improper Output Handling | Medium | Model output is filtered by Content Safety before reaching a consumer ([ADR-0004](../adr/0004-content-filtering-strategy.md)); output is never passed to a downstream execution context without validation — this becomes directly relevant once tool-calling is added ([ADR-0007](../adr/0007-tool-calling-permission-boundary.md)) |
| LLM06 | Excessive Agency | High (forward-looking) | No tool-calling exists in this pattern's baseline scope. Any future addition must comply with per-call scoped permissions and a human-approval gate for mutating actions ([ADR-0007](../adr/0007-tool-calling-permission-boundary.md)) |
| LLM07 | System Prompt Leakage | Medium | No secrets or credentials are placed in the system prompt; the managed identity model ([ADR-0003](../adr/0003-identity-and-rbac-model.md)) means authentication never depends on prompt-embedded values |
| LLM08 | Vector and Embedding Weaknesses | Not applicable | This pattern does not implement retrieval-augmented generation or a vector store. Flagged as a candidate consideration for a future portfolio project that adds RAG |
| LLM09 | Misinformation | Low | Output accuracy and hallucination are primarily a product/UX-layer concern for the consuming application, not an infrastructure control this pattern addresses directly |
| LLM10 | Unbounded Consumption | Medium | Private endpoint deployment ([ADR-0002](../adr/0002-private-networking-pattern.md)) removes public-internet abuse vectors; token/cost monitoring ties into the budget guardrail patterns from [azure-cloud-governance-finops-platform](https://github.com/oyeniffy/azure-cloud-governance-finops-platform) rather than being re-implemented here |

## Explicitly out of scope
- **LLM04 (Data and Model Poisoning)** and **LLM08 (Vector and Embedding
  Weaknesses)** are marked not applicable because this pattern's baseline scope
  includes neither fine-tuning nor RAG. Forcing a mitigation for either here would
  misrepresent what this architecture actually does.
- **Application-layer misinformation controls (LLM09)** are a product decision for
  whatever consumes this pattern, not an infrastructure-layer control.

## Relationship to other portfolio repos
- [multicloud-zero-trust-landing-zone](https://github.com/oyeniffy/multicloud-zero-trust-landing-zone)
  provides the identity and networking substrate this pattern builds on.
- [azure-cloud-governance-finops-platform](https://github.com/oyeniffy/azure-cloud-governance-finops-platform)
  provides the cost-anomaly and budget-guardrail mechanism referenced under LLM10.
