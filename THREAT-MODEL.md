# Drawbridge Threat Model

## Purpose

This document defines the current Drawbridge compromise assumptions, security boundaries, and blast-radius requirements. A fully compromised trust authority is treated as a trust collapse for the systems that depend on it.

## Core invariants

- Device certificate valid does not mean Device authorized.
- Device authorized does not mean User authorized.
- User authorized does not mean Resource authorized.
- Authenticated Device traffic is not necessarily benign traffic.
- Agent request does not mean Gateway authorization.
- Front Distributor placement does not mean Gateway authorization.
- Gateway authorization does not mean enterprise firewall authorization.
- Domain identity does not mean domain administrative authority.
- Drawbridge administration does not mean endpoint, domain, PKI, or firewall administration.
- Tenant Administrator does not mean Site Administrator.
- Trusted Network status cannot be established by SSID, subnet, DHCP, DNS, or gateway address alone.
- Historical validity does not override current revocation, expiration, supersession, or minimum-security policy.
- Commercial license state is not a security authorization input.
- The canonical Record object is immutable from birth.

## Deployment trust modes

### Domain-Managed Mode

The hosting organization owns/administers the Device. The Domain Agent establishes Drawbridge networking and presents the Device credential. Windows and configured identity systems perform User authentication and MFA. The Domain Agent runs under a narrowly scoped gMSA.

### Shared-Service Mode

The participating agency owns/administers the Device. It is not required to join the hosting organization's AD. Shared-service access does not create domain trust, local administrator rights, endpoint-management rights, or general hosting-network access.

The Shared-Service Agent uses an explicit Device certificate/enrollment. User authentication uses the configured protected identity path.

The two Agent modes are distinct top-level implementations/contracts. Shared narrow libraries are permitted; scattered runtime mode checks are not the architecture.

## Agent privilege and compromise

The Agent establishes/maintains networking and presents the Device credential while carrying required Windows/User authentication and MFA traffic.

The Agent is read-oriented and writes only Agent-owned state, exact required network-stack operations, and defined protocol operations.

If elevation is required, a narrow local Network Helper exposes only exact network functions. It is not a generic shell, PowerShell host, filesystem/registry service, or remote-administration mechanism.

Agents do not trust, manage, authenticate to, or directly control peer Agents.

Compromise of one Agent may expose that Device's runtime authority and local Agent state but must not automatically expose peer Devices/Agents, domain administration, Gateway/Controller administration, PKI administration, or production policy authority.

## Lost or stolen Device revocation

Any authorized in-scope Revocation Operator may immediately make a Device revocation authoritative without waiting for a second approver, AD change, normal change window, or certificate-expiration cycle.

Every reachable enforcement point treats revocation as security-urgent state: terminate affected active Device Sessions, deny resume, and deny new sessions. Certificate revocation is initiated and reconnect attempts are recorded.

A genuinely partitioned enforcement point cannot enforce state it has not yet received. Stale authority during a real communications partition is an explicitly bounded distributed-systems risk; once authoritative revocation state becomes reachable, it must take precedence over prior authorization.

A lost/stolen credential is never simply unrevoked. A recovered Device requires hands-on IT verification, new key/CSR, full certificate re-issue, renewed enrollment binding, validation, and return-to-service authorization. The old certificate remains permanently revoked.

Every step is a separate immutable-from-birth canonical Record.

## Front Distributor boundary

The Front Distributor is a traffic-placement component, not an authorization authority.

A compromised Front Distributor may disrupt service, drop/misroute transport, influence Gateway placement, and observe transport metadata available at its layer.

It must not be able to manufacture valid Device/User/Tenant/Resource authorization, create production policy, administer PKI/AD, gain endpoint administration, or replace the Gateway's independent session/authorization checks.

Front Distributor identities/configuration are scoped to ingress placement/health. Individual Front Distributor nodes should be replaceable without changing endpoint configuration.

Total Front Distributor failure follows the Device's signed/versioned Gateway Placement Profile. A profile may fail closed, use an explicitly configured secondary ingress, or permit direct fallback only to explicitly prepared/named Gateways. Failure never authorizes crossing between Domain-Managed and Shared-Service placement profiles, and direct fallback does not weaken Gateway peer authentication or Device/User/Tenant/Resource authorization.

## Gateway boundary

Each Gateway has its own host-specific gMSA set. Functions with different permissions use separate gMSAs where that reduces blast radius. Gateway identities have no general domain or endpoint administrative authority.

Compromise of GW-01 may expose traffic/sessions handled by GW-01, its local enforcement/network authority, local state, host-specific service identities, and Gateway-originated Records integrity. It must not automatically expose peer Gateway credentials, Agent identities, Device private keys, CA signing keys, Controller administration, domain administration, endpoint administration, or enterprise firewall administration.

A compromised Device may generate malicious traffic. The Gateway constrains reachability; it is not an EDR. The enterprise firewall remains an independent deny boundary.

A compromised Gateway can be isolated by revoking its Drawbridge identity, disabling only its host-specific gMSAs, removing it from HA/routing, and isolating network access.

## Controller boundary

The Controller is control plane, not User packet path.

Managed components establish authenticated persistent control connections to the Controller. The Controller logically pushes declarative, versioned artifacts over those established channels. It does not obtain a general-purpose bidirectional remote-administration handle.

Targets verify source, target, schema, version, hash, and authorization; apply locally; verify effective state; and return execution receipts. Status distinguishes sent, received, validated, applied, verified, and failed.

The control protocol has no generic command execution, PowerShell, arbitrary file access, or unrestricted service control.

The Controller should reside on a dedicated management/control network. Each Controller host has host/function-scoped identities.

Controller self-review compares observed security configuration with an explicit baseline and provides remediation guidance without silently broadening permissions or rewriting external systems.

## Records boundary

Records is receive-oriented historical storage/analysis, not live authorization.

The canonical Records object is immutable from birth: complete creation, validation/hash, one durable write, stored-object verification, then no append, patch, or rewrite.

Corrections and later facts create new objects. Mutable indexes/views are derived and rebuildable.

Producer local state is retained until durable acknowledgement of the exact object/hash. Same object ID + same hash is idempotent retransmission; same ID + different content is a security event.

## Service Connector boundary

A Service Connector is a narrowly scoped Resource bridge, not a general internal gateway, jump host, or administrative relay.

It reaches only assigned Resources/supporting services and should be independently constrained by enterprise firewall policy.

Connector identities are host/function scoped. Compromise must not automatically provide general internal-network access, domain administration, Drawbridge policy/enrollment authority, peer-Connector authority, or unrelated Tenant/Resource access.

## Administrative authority

Drawbridge administrative authorization follows the scoped authority model used by Iron Signal Systems Domain Neutral Platform rather than simple role-equals-permissions RBAC.

Human-readable roles may bundle common responsibilities, but authorization depends on a current applicable Authority Grant evaluated against exact identity, Site, Tenant, operation, protected target, governed scope, effective period, policy, and separation-of-duties requirements.

Holding authority and exercising authority are separate facts. Delegation cannot exceed the delegator's current authority.

Revocation authority can rapidly remove trust but does not imply enrollment, recovery, policy, certificate-issuance, or privilege-granting authority.

## Full domain compromise

Full AD domain compromise is collapse of that domain's trust boundary.

Drawbridge does not claim continued trust in domain Users, computer accounts, gMSAs, Kerberos, AD groups, GPO, domain-derived administrator identities, or domain-joined Windows hosts that depend on that domain.

Least-privilege gMSAs contain ordinary host/credential compromise; they are not claimed to survive compromise of the authority governing them.

Independent trust domains remain independent only where they truly do not depend on the compromised domain.

## Drawbridge Recovery Store

The DRS preserves independently verifiable known-good Gateway/Controller configuration outside production Windows/AD trust.

Systems push complete normalized recovery objects after each production configuration promotion/change and at least every 12 hours.

DRS does not pull administrative access from production systems. It is not domain joined.

Preferred design: extremely hardened FreeBSD, ZFS, PF, and separate VNET jails for ingestion, recovery, and optional replication. No network-facing jail has mutable canonical-store access.

A host-side Sealer with no network listener validates staged complete objects, independently hashes them, creates/verifies the canonical object, and seals it.

Canonical DRS objects are immutable from birth. DRS is recovery anchor, not live policy authority.

DRS stores recoverable configuration, not reusable runtime secrets. Recovery objects must not contain private keys, passwords, bearer tokens, reusable MFA material, CA signing keys, software-signing keys, or recoverable gMSA secrets.

Any production configuration required to reconstruct a Drawbridge component must either be deterministically regenerable from DRS-protected authoritative configuration or be preserved in DRS itself.

## PKI compromise

A valid certificate is an identity input, not complete authorization.

Once an issuing PKI authority is declared compromised, no new trust establishment through it is permitted: new Device connections, reconnects requiring fresh authentication, initialization/bootstrap, enrollment/renewal, and affected infrastructure trust establishment are denied.

The compromise decision is authoritative immediately. Components that have received that state fail closed for new trust; a genuinely partitioned component cannot enforce state it has not yet received and must revalidate current trust when communication is restored.

Existing established sessions are a separate incident-response decision.

Revoked security credentials are replaced with new keys/certificates rather than reactivated.

## Hostile network and DNS

Transport networks are assumed hostile-capable. IP, DHCP, DNS, SSID, subnet, gateway address, and apparent enterprise topology do not prove trust.

DNS is discovery/resolution, not authorization. Cryptographic peer authentication proves Drawbridge infrastructure identity. DNS answers do not independently expand Resource authorization.

Transport path/IP changes do not change Device identity. Session continuity remains cryptographically bound.

IPv4 and IPv6 must be governed consistently.

## Replay and downgrade

Historical validity does not create current authority. Current revocation always overrides earlier authorization.

Production configuration versions advance monotonically. Intentional rollback creates a new tested/promoted version representing the rollback; enforcement components do not simply activate an older production artifact.

Protocol, schema, certificate-profile, and cryptographic negotiation enforce minimum accepted versions and do not silently fall back.

## Software, update, and supply-chain compromise

Executable-software authority is separate from runtime configuration authority.

The normal Controller control channel may distribute declarative configuration artifacts but must not become a generic mechanism for arbitrary binary installation or code execution.

Production updates are tied to:

- exact GitHub source commit pin;
- component purpose and target platform;
- release/version identity;
- strong artifact hash;
- signed ISS release manifest;
- current release authorization/minimum-version policy.

Floating Git references such as `main`, `HEAD`, or `latest` are not production update identities.

GitHub provides source/release provenance but is not the sole production software trust root. A production component must be able to verify an approved artifact from its signed manifest and artifact hash without requiring live GitHub availability at installation time.

Source-control access, build authority, release approval, software-signing authority, distribution, and production installation are distinct security boundaries. Runtime Agent/Gateway/Controller/Connector/DRS identities do not possess production software-signing keys.

Historical signature validity does not override current software-release revocation or minimum accepted version. Compromise of the production software-signing authority is a software trust collapse for releases beneath that authority.

## Availability and denial of service

Unavailable and compromised are different states.

Loss of a non-forwarding dependency does not automatically invalidate previously established authorization that remains independently current. In particular, temporary Controller, Records, or DRS unavailability must not unnecessarily terminate valid established forwarding.

Fresh security decisions fail when their required authority is unavailable and no explicitly defined bounded cached decision remains valid. Drawbridge does not silently bypass required authentication, MFA, PKI, policy, or identity stages to preserve apparent connectivity.

Front Distributor failure should fail over through the stable Drawbridge Service Address without changing Device identity or silently broadening authorization.

Gateway failure should fail over/resume through another authorized Gateway without changing Device identity.

Queues, pending control work, spools, staging areas, session counts, and storage are bounded. Resource controls should isolate one Device, producer, or Tenant from exhausting the entire Site where practical.

Security-urgent state such as Device revocation or compromised-issuer state is prioritized over routine configuration and health traffic.

DRS unavailability degrades recovery readiness and health status; it is not by itself a reason to stop otherwise authorized public-safety forwarding.

## Privacy and sensitive telemetry

Drawbridge collects telemetry for defined operational, security, troubleshooting, policy-verification, audit, and recovery purposes.

Device location is a first-class recorded observation where the platform can provide it. Location Records preserve source, timestamp, freshness/age, and accuracy where available. Physical location and network-derived location are distinct facts, and location is an observation rather than infallible proof of position.

Routine Drawbridge Records do not contain application payload bodies, passwords, private keys, bearer tokens, reusable authentication secrets, MFA secrets, or other reusable credential material.

Process attribution is initially observational and must not be represented as stronger than the underlying platform observation supports.

Records authorization is separate from policy/system/storage administration. Tenant/Site scope is enforced by backend authorization. Sensitive queries, bulk exports, retention changes, and governed destruction are attributable. Immutability does not mean indefinite retention.

## Accepted risks and explicit non-goals

Drawbridge reduces and compartmentalizes risk; it does not make a compromised endpoint, compromised trust authority, malicious authorized User, vulnerable application, or unavailable network safe.

A fully compromised Device may generate malicious traffic and may falsify Device-originated observations. Gateway/resource/firewall boundaries remain independent controls.

Immutable history protects already committed canonical objects from later modification through normal producer authority; it does not make a compromised producer truthful about new observations.

A fully compromised Windows trust domain, PKI issuer, software-signing authority, Gateway host, Controller host, or DRS host invalidates the guarantees that depend on that authority. A compromised Front Distributor has serious availability/traffic-placement impact but does not by design become an authorization authority. The design objective is blast-radius containment and independent recovery, not magical preservation of trust after the governing authority is owned.

An authorized administrator inherently possesses the ability to exercise the authority actually granted, including potentially disruptive actions such as in-scope revocation or production change.

Drawbridge cannot create network availability where no viable transport exists and cannot force an upstream network to forward traffic. Encrypted transport does not make Drawbridge an anonymity system; underlying networks may still observe metadata such as endpoint/Gateway addresses, timing, volume, and duration.

Drawbridge is not an EDR, DLP system, CASB, SWG, general enterprise firewall, general RMM platform, malware sandbox, full SIEM replacement, application-security replacement, identity-provider replacement, or PKI replacement.

Commercial licensing state is never a security authorization input or packet-forwarding kill switch.

## Threat-model baseline status

Issue #2 threat-model design baseline is closed. Exact timeout values, cache lifetimes, rate limits, failover thresholds, and other implementation constants remain owned by their corresponding later design issues rather than being guessed here.
