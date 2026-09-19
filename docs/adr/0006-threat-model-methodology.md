# ADR-0006: Threat Model Methodology

## Status
Accepted

## Context
This repo's differentiator is a documented threat model specific to LLM integration,
not just a description of what was built. A threat model needs a taxonomy — a
structured way to enumerate risk categories — rather than an ad hoc list of concerns.
The two realistic options are an internally invented taxonomy tailored to this specific
architecture, or an established industry-recognized standard.

## Decision
Use the **OWASP Top 10 for LLM Applications** as the baseline taxonomy for the threat
model (`docs/threat-model/llm-threat-model.md`), mapping each of the ten risk
categories against this architecture explicitly, including categories judged low or
non-applicable.

## Rationale
- **Checkable against a public standard.** A reviewer — a hiring panel, a client
  security team — can hold the threat model's claims up against a taxonomy they
  already recognize, rather than having to trust an architecture-specific framing they
  cannot independently verify. This is a credibility choice, not just a convenience.
- **Forces completeness.** Working through all ten categories, rather than only the
  ones that come to mind first, surfaces gaps that an ad hoc list would miss — this ADR
  and the resulting threat model document explicitly mark categories as low-relevance
  or out of scope rather than silently omitting them.
- **Maps cleanly onto decisions already made.** Several categories in the OWASP
  taxonomy correspond directly to earlier ADRs in this repo — prompt injection and
  system prompt leakage relate to ADR-0004's content filtering scope, excessive agency
  relates to ADR-0007's tool-calling boundary, sensitive information disclosure relates
  to ADR-0005's logging design. Using this taxonomy makes those connections explicit
  rather than requiring the reader to infer them.

## Trade-offs and what we're giving up
- **Not every category applies equally.** Categories like data/model poisoning and
  vector/embedding weaknesses are largely inapplicable to a pattern that does not
  fine-tune models or implement retrieval-augmented generation. The threat model marks
  these explicitly as low-relevance with a stated reason, rather than forcing an
  artificial mitigation to appear thorough — a reviewer familiar with the standard will
  recognize a forced mitigation as weaker signal than an honest "not applicable, here's
  why."
- **The standard evolves.** OWASP's LLM taxonomy has been revised since its initial
  publication and will likely be revised again. This ADR pins the version used at the
  time of writing; a future maintainer should verify whether a newer version changes
  category definitions materially before treating this document as current.

## Consequences
- `docs/threat-model/llm-threat-model.md` is structured as a table mapping all ten
  OWASP LLM risk categories to this architecture's relevance and mitigation (or
  explicit non-applicability).
- Categories judged non-applicable are documented with a stated reason, not omitted
  silently.
- Future ADRs in this repo (0007, and any later additions) are expected to reference
  the specific OWASP category they address, keeping the mapping traceable.
