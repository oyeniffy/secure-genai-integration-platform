# ADR-0005: Audit Logging and Data Retention

## Status
Accepted

## Context
The business problem this repo solves is driven by an audit and compliance
requirement: security teams need to reconstruct what was asked, what was returned, and
by whom, for any interaction with the model. The naive approach — log every prompt and
completion in full, indefinitely — satisfies the audit requirement but creates a second
problem: the audit log itself becomes a store of potentially sensitive or confidential
data, with its own access-control and data-residency exposure. This is a genuine
tension, not a solved problem, and this ADR states the position taken rather than
hiding the trade-off.

## Decision
Log **structured metadata** for every request by default — correlation ID, identity
(from ADR-0003's managed identity), timestamp, token counts, model deployment name, and
the Content Safety filter verdict (ADR-0004) — to the standard Log Analytics workspace,
forwarded to Sentinel for alerting. **Full prompt and completion content** is logged
only to a separate, higher-restriction workspace with tighter RBAC scope, treated as a
break-glass capability for incident investigation rather than a default-on setting.

## Rationale
- **Metadata satisfies most audit questions without content exposure.** Who called the
  model, when, how much was processed, and whether the content filter flagged
  anything — this answers the majority of compliance and anomaly-detection questions
  (e.g. unusual volume, off-hours access, repeated filter triggers) without ever storing
  the actual conversation content.
- **Full-content logging as a separate, restricted capability avoids creating a second
  exposure surface by default.** If full prompt/response content were logged
  everywhere by default, the audit log itself becomes a target — anyone with read
  access to "the logs" would have read access to every sensitive conversation that ever
  passed through the system. Restricting that to a break-glass workspace keeps the
  blast radius of a logging-workspace compromise limited to metadata.
- **Correlation ID enables reconstruction without duplication.** If a genuine
  investigation requires full content, the correlation ID from the metadata log links
  to the break-glass workspace's record — the two are joined only when someone with the
  elevated access deliberately does so, not by default.

## Trade-offs and what we're giving up
- **Metadata alone cannot answer every audit question.** "What exactly did the model
  say in this specific interaction" requires the break-glass workspace — metadata-only
  logging is a deliberate limitation on day-to-day visibility, in exchange for reduced
  default exposure.
- **Break-glass access itself needs governance this repo does not fully specify.** Who
  can access the full-content workspace, under what approval process, and for how long,
  is an organizational control this ADR flags as necessary but does not design in
  detail — that is client-specific policy, not an infrastructure decision.
- **Retention period is a genuine open question, not resolved here.** Regulatory
  retention requirements vary by jurisdiction and industry (see ADR-0008 for the
  residency dimension of this). This repo defaults both workspaces to Log Analytics'
  standard retention setting and flags that a real deployment must set this explicitly
  against the client's actual regulatory obligation, not leave it at a platform default
  by omission.

## Consequences
- The Bicep module (`infra/bicep/modules/log-analytics-sentinel.bicep`) provisions two
  workspaces (or one workspace with two tables under different RBAC scopes) rather
  than a single undifferentiated log store.
- The threat model documents this as the mitigation for LLM02 (Sensitive Information
  Disclosure) as it applies to the logging layer itself, not just the model's output.
- Any future extension of this pattern (e.g. a SIEM integration) should preserve the
  metadata/full-content separation rather than collapsing it back into one stream.
