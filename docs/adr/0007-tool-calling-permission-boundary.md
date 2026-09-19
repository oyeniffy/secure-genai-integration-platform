# ADR-0007: Tool-Calling Permission Boundary

## Status
Accepted

## Context
Enterprise GenAI workflows increasingly extend beyond plain chat completion into
tool-calling or function-calling, where the model can invoke actions against other
systems (querying a database, calling an internal API, triggering a workflow). This is
the OWASP LLM06 (Excessive Agency) risk category: if a model's tool access is scoped
too broadly, a successful prompt injection (LLM01) can escalate from "the model said
something wrong" to "the model did something wrong" — a materially worse outcome. This
pattern does not require tool-calling for its baseline chat/completion scope, but any
enterprise adoption of this pattern is likely to add it, so the permission boundary
needs to be decided now rather than retrofitted under pressure later.

## Decision
Any tool-calling capability added to this pattern must be **scoped per-call, not
per-service**, using the same managed identity model from ADR-0003,
and any tool call
that **mutates state** (writes, updates, deletes, sends) requires a **human-approval
gate** before execution. Read-only tool calls (lookups, queries) may execute without a
gate but are still logged per ADR-0005.

## Rationale
- **Per-call scoping limits blast radius.** Granting the calling identity broad access
  "because the model might need it" is the same anti-pattern as over-permissioned
  service accounts elsewhere in cloud architecture — the fix is the same: grant only
  the specific action needed for the specific call, not a standing broad grant.
- **Human approval for mutations is the correct trade against Excessive Agency.** A
  prompt injection that convinces the model to attempt a mutating action is contained
  by the same control that would catch a legitimate mistaken action — a human decision
  point before anything changes state. This is a stronger control than trying to
  detect malicious intent computationally, which is an unsolved problem.
- **Read-only calls without a gate keep the pattern usable.** Requiring approval for
  every single tool call, including harmless lookups, would make the pattern
  impractical to demonstrate or adopt. The mutating/non-mutating distinction is the
  place to draw the line, not an all-or-nothing gate.

## Trade-offs and what we're giving up
- **Human-approval gates break full automation.** Any workflow that requires the
  mutating action to happen without a human in the loop cannot use this pattern as
  designed — that is a deliberate constraint, not an oversight, and a client wanting
  fully autonomous mutating actions needs a different risk conversation entirely, not a
  configuration change to this pattern.
- **The mutating/non-mutating distinction requires per-tool classification effort.**
  Every tool added to the system must be explicitly classified as mutating or
  read-only at design time — this is manual classification work, not something the
  platform infers automatically, and a misclassified tool undermines the entire
  control.
- **This ADR is forward-looking.** No tool-calling implementation exists in this
  repo's baseline scope; this decision establishes the boundary that any future
  tool-calling addition must comply with, rather than describing an already-built
  capability.

## Consequences
- Any future tool integration in this repo must document, per tool, whether it is
  read-only or mutating, and mutating tools must integrate an approval step before
  execution.
- The threat model's LLM06 entry references this ADR as the stated mitigation.
- The identity and RBAC model (ADR-0003) extends to tool-scoped permissions rather than
  a single blanket grant when tool-calling is added.
