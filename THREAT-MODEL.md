# Drawbridge Threat Model

## Purpose

This document defines the current Drawbridge compromise assumptions, security boundaries, and blast-radius requirements. A fully compromised trust authority is treated as a trust collapse for the systems that depend on it.

## Core invariants

- Device certificate valid does not mean Device authorized.
- Device authorized does not mean User authorized.
- User authorized does not mean Resource authorized.
- Authenticated Device traffic is not necessarily benign traffic.
- Agent request does not mean Gateway authorization.
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

Any authorized in-scope Revocation Operator may immediately revoke a Device without waiting for a second approver, AD change, normal change window, or certificate-expiration cycle.

Revocation terminates active Device Sessions and denies resume/new sessions. Certificate revocation is initiated and reconnect attempts are recorded.

A lost/stolen credential is never simply unrevoked. A recovered Device requires hands-on IT verification, new key/CSR, full certificate re-issue, renewed enrollment binding, validation, and return-to-service authorization. The old certificate remains permanently revoked.

Every step is a separate immutable-from-birth canonical Record.

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

## PKI compromise

A valid certificate is an identity input, not complete authorization.

Once an issuing PKI authority is declared compromised, no new trust establishment through it is permitted: new Device connections, reconnects requiring fresh authentication, initialization/bootstrap, enrollment/renewal, and affected infrastructure trust establishment are denied.

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

## Remaining work

Before this threat-model issue is closed:

- software/update and supply-chain compromise;
- availability and denial-of-service boundaries;
- privacy/sensitive telemetry boundaries;
- final accepted-risk and non-goal statements.
