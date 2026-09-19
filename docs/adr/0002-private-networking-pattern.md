# ADR-0002: Private Networking Pattern

## Status
Accepted

## Context
ADR-0001 established Azure OpenAI Service as the LLM platform. The business problem
this repo solves explicitly requires no public endpoint exposure — security and
compliance sign-off depends on the model endpoint being unreachable from the public
internet. Azure OpenAI is a Cognitive Services resource, which constrains the private
networking options available: it does not support VNet injection (unlike an App Service
or VM), so the choice is effectively between a Private Endpoint pattern and accepting
public exposure with IP allow-listing as a lesser control.

## Decision


Deploy Azure OpenAI with the public network access flag disabled, exposed only via a
**Private Endpoint** connected to the hub VNet established in
[multicloud-zero-trust-landing-zone](https://github.com/oyeniffy/multicloud-zero-trust-landing-zone),
with a **Private DNS Zone** (`privatelink.openai.azure.com`) linked to that same hub
VNet to resolve the resource's FQDN to its private IP.

## Rationale
- **No public endpoint, by construction.** Disabling public network access at the
  resource level is a stronger control than IP allow-listing, which is a compensating
  control, not a structural one — an allow-list can be bypassed by anything that spoofs
  or transits an allowed IP; a disabled public endpoint has no such surface.
- **Reuses existing hub-spoke topology.** The landing zone repo already establishes a
  hub VNet and private DNS zone pattern for other Azure PaaS services. Extending that
  hub, rather than building a parallel private-networking story for this repo, keeps
  the portfolio's architecture internally consistent.
- **Consuming services must already be network-integrated.** Anything calling the
  Azure OpenAI endpoint (an API, a Function App, a container) must itself be inside the
  VNet, peered to it, or reachable via VPN/ExpressRoute — this is a deliberate forcing
  function that keeps the entire request path private, not just the model endpoint.

## Trade-offs and what we're giving up
- **Added topology complexity for consumers.** Any client of this endpoint that isn't
  already network-integrated (e.g. a developer's local machine, a SaaS integration
  outside the VNet) cannot reach it directly and needs a bastion, VPN, or a
  network-integrated intermediary. This is a real operational cost, not a hidden one.
- **DNS resolution is a common failure mode.** If the Private DNS Zone is linked to the
  wrong VNet (e.g. the spoke instead of the hub, or not linked at all), the resource
  FQDN resolves to its public IP or fails to resolve entirely — and because the public
  endpoint is disabled, this fails as a connection timeout rather than an obvious DNS
  error, which is harder to diagnose. This is documented explicitly here because it is
  the single most common real-world misconfiguration with this pattern.
- **No support for VNet injection.** Unlike compute resources, Cognitive Services
  resources cannot be injected directly into a subnet — Private Endpoint is not a
  fallback choice here, it is the only structural option, which simplifies this
  decision but should not be mistaken for a broader private-networking capability.

## Consequences
- Any service consuming this Azure OpenAI endpoint (documented in later ADRs and the
  architecture overview) must be network-integrated with the hub VNet.
- The Private DNS Zone linkage is called out explicitly as a deployment verification
  step in the IaC and in any operational runbook for this repo.
- Diagnostic settings (ADR-0005) route through the same private networking boundary,
  avoiding a second public-exposure decision for logging.
