# Secure GenAI Integration Platform

## Business problem
An enterprise — a bank, a telco — wants to adopt GenAI for a real workflow, but
security and compliance will not sign off without three guarantees: data residency,
a documented defence against prompt injection and data exfiltration, and full audit
logging of every prompt and response. This repository demonstrates a pattern that
satisfies all three, using Azure OpenAI deployed behind a private endpoint, accessed
via managed identity, with content filtering and structured audit logging.

## Architecture

| Component | Azure Service | Purpose |
|---|---|---|
| Model endpoint | Azure OpenAI Service | Hosts the LLM, no public endpoint |
| Network boundary | Private Endpoint + Private DNS Zone | Removes public-internet exposure entirely |
| Identity | User-assigned managed identity + custom RBAC role | Authenticates callers without standing credentials |
| Content filtering | Azure AI Content Safety | Filters prompts and completions, detects injection attempts |
| Audit logging | Log Analytics + Microsoft Sentinel | Metadata logged by default; full content gated behind a restricted workspace |
| Data residency | Azure Policy (region restriction) | Enforces residency as an enforced control, not a documentation promise |

Full component rationale and trade-offs: [architecture overview](docs/architecture/architecture-overview.md).
Every decision behind this table is documented in [`docs/adr/`](docs/adr/), numbered
0001 through 0008.

## Threat model
This repository includes a documented threat model mapped to the OWASP Top 10 for LLM
Applications, covering prompt injection, sensitive information disclosure, excessive
agency, and the rest of the taxonomy — including categories explicitly marked as
non-applicable, with the reasoning stated rather than omitted.
See [`docs/threat-model/llm-threat-model.md`](docs/threat-model/llm-threat-model.md)
for the full mapping.

## Status
Architecture decisions, the threat model, and the architecture overview are complete
and documented. Infrastructure-as-code is in progress, validated via
`bicep what-if` before any live deployment. Live deployment and adversarial testing
against the threat model are staged for a deliberate budget window, following the same
staged-deployment discipline applied in
[resilient-postgres-platform](https://github.com/oyeniffy/resilient-postgres-platform).

## Related work
This pattern builds on the identity and networking foundation established in
[multicloud-zero-trust-landing-zone](https://github.com/oyeniffy/multicloud-zero-trust-landing-zone),
and its cost and budget controls extend the patterns in
[azure-cloud-governance-finops-platform](https://github.com/oyeniffy/azure-cloud-governance-finops-platform).

## License
[MIT](LICENSE)
