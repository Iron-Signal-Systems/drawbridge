# Open Questions

These items are intentionally unresolved and should be decided before or during Phase 0.

## Transport

- QUIC, CONNECT-IP over HTTP/3, or another framing model?
- How much reliability is provided by the transport versus Drawbridge session logic?
- How are packet replay/reordering and dead-zone recovery bounded?
- Is multipath a future extension or a v1 consideration?

## Virtual addressing

- Stable per device, per device certificate, or per logical session?
- Separate address pools per tenant?
- Route advertisement model?
- How are overlapping agency RFC1918 networks represented?

## Windows enforcement

- Exact use of WFP?
- Virtual adapter implementation?
- Which functions require kernel-mode components versus privileged user-mode services?
- How are SYSTEM/service traffic and interactive-user traffic distinguished?

## Trusted-network proof

- Which signals are mandatory?
- Can EAP-TLS/802.1X state be consumed reliably?
- Is an authenticated on-network Drawbridge responder required?
- How are false trusted classifications prevented?

## Directory synchronization

- Preferred AD incremental sync mechanism?
- Nested group semantics?
- Multiple domains/forests?
- Read-only gMSA permission model?
- Maximum acceptable stale-directory interval?

## Shared services

- Federation contract for OIDC/SAML?
- RADIUS proxy behavior?
- Device enrollment approval workflow?
- Tenant delegated administrators?
- Local identity requirements and MFA baseline?

## Records

- Canonical schema/encoding?
- Local endpoint spool format?
- Ingestion protocol?
- Signing/checkpoint model?
- Default retention?
- Physical location default: off?
- Process attribution coverage and privacy boundaries?

## HA

- Replicated session state versus resumable cryptographic token?
- Gateway selection?
- Site affinity?
- Geographic redundancy?
- Data-plane behavior if Controller is unavailable for extended periods?

## Routing and service handoff

- Static routes only for v1, or dynamic routing support?
- Which dynamic routing protocol(s), if any, are justified later?
- Exact NAT compatibility behavior?
- Service proxy protocol coverage: TCP only first, or generic IP/UDP support?
- Should the internal Service Connector initiate its channel outward to avoid DMZ-initiated connections?
- How are overlapping tenant address spaces represented without losing audit identity?
- How are private DNS namespaces and FQDN-to-address bindings enforced securely?

## Agent sensor and DNS implementation

- Exact Windows capture/enforcement points for DNS and socket creation?
- WFP versus Windows DNS policy/NRPT integration versus hybrid approach?
- How should process identity be preserved when applications use shared resolver services?
- Exact DoH/DoT detection/control model?
- How are CNAME chains and multi-address responses bound to policy?
- How are CDN/FQDN address changes handled without over-broad temporary authorization?
- What is the safe default if a protected namespace resolver is unavailable?

## DNS

- Split-DNS architecture?
- FQDN routing refresh/TTL semantics?
- How are historical DNS observations linked to later connections?
- How are private tenant zones handled?

## Licensing

Site licensing is the design direction, but commercial tiers still need to be defined:

- site class based on throughput?
- number of production Gateway clusters?
- support SLA?
- regional/multi-site pricing?
- hosted-service pricing?

Regardless of commercial model, licensing must remain outside the production forwarding decision.

## Legal/IP

Before serious product investment:

- perform freedom-to-operate patent review;
- review third-party protocol/library licenses;
- define cryptographic compliance targets;
- define CJIS/FIPS positioning carefully and accurately.

## Pilot acceptance

Need an explicit acceptance matrix for:

- PD Ops;
- 911/EMO;
- network administration;
- systems/PKI;
- security;
- application owners.

The pilot success definition must be operational, not merely "tunnel connected."
