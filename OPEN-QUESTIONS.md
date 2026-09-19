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
- Exact read attributes and delegation required by each host/function-specific gMSA?
- Maximum acceptable stale-directory interval?

## Shared services

- Federation contract for OIDC/SAML?
- RADIUS proxy behavior?
- Device enrollment approval workflow?
- Tenant delegated administrators?
- Local identity requirements and MFA baseline?

## Records

The canonical object model is decided: complete write-once objects are immutable from birth.

- Canonical schema/encoding?
- Local endpoint spool format?
- Ingestion protocol and durable-receipt format?
- Signing/checkpoint model?
- Default retention?
- Exact location-source precedence, accuracy/freshness semantics, and retention profile?
- Process attribution coverage and privacy boundaries?

## Controller control plane

- Exact authenticated persistent-channel transport?
- Exact artifact signing/verification profile?
- How are urgent revocations prioritized over routine configuration distribution?
- How is effective-state verification represented and retried?
- How is the dedicated Controller management/control network segmented for small versus large sites?

## Recovery Store

Core architecture is decided: independent non-domain storage, immutable-from-birth complete snapshots, immediate snapshot after production change, periodic snapshot at least every 12 hours, hardened FreeBSD/ZFS/PF/VNET preferred.

Open implementation questions:

- physical appliance versus independently administered VM profile?
- exact ZFS dataset/object layout?
- independent administrative authentication profile?
- secondary/off-site replication target?
- signing/checkpoint/witness profile?
- maximum accepted checkpoint age?

## HA

Core architecture is decided:

- Agents normally target a stable Drawbridge Service Address rather than named Gateways;
- production uses a redundant Drawbridge Front Distributor tier;
- Front Distributors perform traffic placement only and do not grant authorization;
- Gateways independently validate/enforce Drawbridge session/authorization state;
- Front Distributor state should be disposable/reconstructable where practical;
- Gateway Placement Profiles bind Device populations/trust domains to primary ingress, eligible Gateway pools, and explicit total Front Distributor failure behavior;
- total Front Distributor failure behavior is profile-scoped: FAIL_CLOSED, SECONDARY_INGRESS, or DIRECT_GATEWAY_FALLBACK;
- direct fallback may use only explicitly prepared/named Gateways with deterministic configured selection/order;
- Domain-Managed and Shared-Service profiles do not cross-select each other's ingress/Gateway pools during failure;
- fallback changes transport placement only and does not weaken normal identity/session/Resource authorization;
- Transport Path, Front Distributor, or Gateway failure does not by itself redefine Device/User identity or the logical Device Session.

Open implementation/ADR questions:

- active/standby versus active/active Front Distributors?
- on-prem Service Address mechanism: CARP/VRRP, anycast, external load balancer, or another design?
- transparent versus transport-aware Front Distributor behavior?
- exact Gateway health/eligibility contract?
- replicated session state versus resumable cryptographic token/state versus hybrid?
- if QUIC is selected, should Connection-ID-aware routing encode/recover Gateway placement?
- exact Gateway Placement Profile schema, signing, distribution, caching, expiry, and update semantics?
- direct-fallback Gateway exposure/health mechanism and deterministic selection algorithm?
- Site affinity and multi-ingress selection?
- geographic redundancy/failure-domain design?
- exact data-plane behavior if Controller is unavailable for extended periods?
- interruption/resume SLO targets?

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


## Software supply chain

The trust model is decided: production releases use an exact GitHub source commit pin, strong artifact hash, signed ISS release manifest, component/platform identity, and current release authorization. GitHub provenance is not the sole runtime trust root.

Open implementation questions:

- exact release-manifest schema and signature profile?
- signing-key protection/HSM or hardware-token workflow?
- release revocation and minimum-version distribution format?
- independently hosted/offline artifact mirror strategy?
- build provenance and reproducibility requirements at each maturity stage?

## Availability thresholds

The fail-behavior principles are decided; exact operational constants remain open:

- maximum policy-cache age by decision class?
- session-resumption window?
- reconnect backoff/jitter parameters?
- per-Device/per-Tenant/session/control queue limits?
- Records/DRS spool thresholds and escalation levels?
- maximum DRS checkpoint age before warning/critical state?

## Threat-model baseline status

Issue #2 threat-model design baseline is closed. Remaining questions above are implementation parameters or later-design ownership, not unresolved threat-model principles.
