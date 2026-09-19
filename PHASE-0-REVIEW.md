# Drawbridge Phase 0 Design Review

## Purpose

This document tracks the remaining pre-code Drawbridge design work in dependency order.

Phase 0 ends with a coherent implementation baseline, not merely a collection of ideas. A design item is considered closed when its security authority, failure behavior, ownership boundary, and implementation-facing contract are sufficiently explicit that Phase 1 code does not need to invent architecture.

## Closed design baselines

### P0-01 - Orthogonal authoritative state model — CLOSED

Locked state domains:

- DeviceSessionState;
- TransportState;
- DeviceAuthorizationState;
- UserAuthorizationState;
- NetworkTrustState;
- PolicyPosture;
- HealthState.

State, reason, and event remain distinct. Transport/path changes do not redefine Device identity or the logical Device Session.

### P0-02 - Threat and compromise model — CLOSED

THREAT-MODEL.md now defines:

- Agent/Gateway/Controller/Connector/Records/DRS compromise boundaries;
- domain and PKI trust collapse;
- lost/stolen Device revocation/recovery;
- hostile transport/DNS assumptions;
- replay/downgrade behavior;
- software/update supply-chain trust;
- availability/DoS principles;
- sensitive telemetry/location handling;
- accepted risks/non-goals.

### P0-03 - Agent / Front Distributor / Gateway / Firewall enforcement boundary — CLOSED

ENFORCEMENT-BOUNDARIES.md now locks:

- Agent DIRECT/TUNNEL/DENY semantics;
- TUNNEL as path selection rather than Gateway authorization;
- traffic-placement-only Front Distributor authority;
- independent Gateway Device/session/User/Tenant/Resource authorization;
- restrictive intersection of Agent and Gateway permits;
- no transient broader grants from policy-generation skew;
- unverified Agent process/location/network/DNS claims cannot independently broaden Gateway authorization;
- independent enterprise firewall/network deny/routing authority;
- explicit return versus unsolicited-inbound behavior;
- Domain-Managed versus Shared-Service bootstrap/enforcement differences;
- independent Agent/Gateway observations and fail-restrictive disagreement handling.

### P0-HA-A - HA ingress topology and authority — CLOSED

HA-AND-SESSION-CONTINUITY.md locks:

- stable Drawbridge Service Address;
- redundant Drawbridge Front Distributor tier;
- traffic-placement-only Front Distributor authority;
- Gateway-independent authorization enforcement;
- disposable/reconstructable Front Distributor state where practical;
- versioned Gateway Placement Profiles defining primary ingress/Gateway pools and explicit total Front Distributor failure behavior;
- profile-scoped FAIL_CLOSED, SECONDARY_INGRESS, or explicitly authorized DIRECT_GATEWAY_FALLBACK behavior;
- deterministic direct-fallback targets/order and no Domain/Shared-Service cross-profile selection during failure;
- identity/session continuity as independent from Transport Path/Front Distributor/Gateway assignment.

The exact session recovery mechanism remains open and is tracked below.

### Other settled Phase 0 contracts

The following already have sufficient design direction and should not be reopened casually:

- Domain Agent versus Shared-Service Agent top-level split;
- narrow Agent Network Helper privilege pattern;
- no Agent-to-Agent trust/control;
- site/deployment licensing outside forwarding decisions;
- DNP-style scoped Authority Grant model as the administrative authorization basis;
- immutable-from-birth canonical Records/recovery objects;
- Drawbridge Recovery Store trust boundary and FreeBSD/VNET preferred architecture;
- exact-artifact test/promotion and monotonic rollback-as-new-version;
- location as a normal first-class recorded observation where available;
- process attribution is observational unless stronger platform proof exists;
- signed update provenance tied to exact GitHub source commit and artifact hash.

## Remaining Phase 0 work — dependency order

### P0-04 - User authorization assertion and User Session binding

Define how successful Windows/AD/NPS/RADIUS/federated authentication becomes a bounded Drawbridge User Session.

Must resolve:

- assertion source/issuer;
- User identity;
- Device Session binding;
- Tenant binding;
- purpose/audience;
- issue/expiry time;
- MFA/authentication context;
- replay prevention;
- refresh/re-authentication;
- logout/lock/switch-user behavior;
- stale/authority-unavailable behavior.

Exit condition: a Gateway can determine exactly why a User Session is current without receiving/replaying User passwords.

### P0-05 - Deterministic policy evaluation pipeline

Freeze the ordered policy decision inputs and conflict behavior.

At minimum:

- Device authorization;
- User authorization where required;
- Tenant;
- Resource;
- network trust;
- PolicyPosture;
- configuration/policy generation;
- current revocation;
- explicit direction;
- enterprise handoff.

Exit condition: the same inputs always produce the same explainable decision; ambiguity is rejected rather than resolved by hidden rule order.

### P0-06 - PKI profiles and credential purposes

Define distinct certificate/key profiles for at least:

- Domain-Managed Device;
- Shared-Service Device;
- Gateway;
- Controller;
- Front Distributor if cryptographic identity is required at that layer;
- Service Connector;
- Records/DRS service identities where appropriate;
- software signing separately from operational PKI.

Freeze EKU/purpose, subject/SAN identity binding, key storage, rotation, revocation, issuer separation, enrollment, and trust-failure behavior.

Exit condition: a valid credential for one purpose cannot silently authenticate another Drawbridge role.

### P0-07 - Records v0 canonical envelope

Freeze the first canonical Record envelope before substantial implementation.

Define:

- record/object ID;
- producer identity;
- producer role;
- Site/Tenant;
- Device/User/session correlation;
- event type;
- observed time and receive/commit time;
- software/configuration version;
- payload/schema version;
- prior/related object references;
- integrity/hash/signature/checkpoint fields;
- explicit unknown/not-observed/not-applicable semantics.

Exit condition: Phase 1 components can emit stable Records without inventing incompatible event envelopes.

### P0-08 - Time and event ordering

Define how Drawbridge reasons about time without pretending all clocks are perfect.

Resolve:

- wall-clock versus monotonic time use;
- clock source/quality;
- skew handling;
- producer sequence numbers;
- ordering within a producer;
- cross-producer correlation;
- receive/commit time;
- session-relative ordering;
- behavior after clock rollback;
- time dependence of expiry/revocation.

Exit condition: event chronology, expiry, and replay protection remain defensible during clock skew/rollback.

### P0-09 - Transport ADR

Select the transport direction after the enforcement, identity, PKI, and Records inputs are known.

Evaluate at least:

- QUIC-based design;
- CONNECT-IP over HTTP/3/QUIC where appropriate;
- another mature protected transport with Drawbridge framing.

The ADR must cover:

- cryptographic peer identity;
- session IDs independent of external IP;
- NAT rebinding;
- path migration;
- replay protection;
- packet ordering/reordering;
- MTU/PMTUD;
- IPv4/IPv6;
- Front Distributor compatibility;
- diagnosability;
- suspend/resume primitives.

Exit condition: Phase 1 can implement one transport path without inventing new cryptography or HA semantics.

### P0-10 - HA session recovery mechanism

The ingress topology and Gateway Placement Profile failure-policy contract are already closed. This issue selects how a logical Device Session survives Gateway failure and resumes when the authorized transport destination changes.

Evaluate:

- replicated/shared Gateway session state;
- cryptographically protected portable resume state/token;
- hybrid design.

Must define:

- resume proof;
- current revocation check;
- User Session binding;
- virtual identity/address ownership;
- policy-generation binding;
- replay resistance;
- resume lifetime;
- Gateway change;
- key rotation;
- failure Records.

Exit condition: Gateway failure can be implemented without making Front Distributor state precious or weakening current authorization.

### P0-11 - Windows Agent architecture

Define the concrete Windows composition for Domain Agent and Shared-Service Agent.

Resolve:

- Windows service boundaries;
- gMSA use in Domain Agent;
- narrow privileged Network Helper;
- WFP ownership;
- virtual adapter model;
- user-mode versus kernel-mode responsibilities;
- IPC ACLs/message contract;
- DNS integration;
- sensor observation points;
- update/install privilege separation;
- EDR coexistence.

Exit condition: implementation can start without a monolithic privileged service.

### P0-12 - Trusted Network proof

Define what positively establishes NetworkTrustState=TRUSTED.

Potential inputs may include:

- EAP-TLS/802.1X state;
- authenticated Drawbridge on-network responder;
- cryptographic enterprise proof;
- supporting network characteristics as secondary signals.

SSID/subnet/DHCP/DNS/gateway address remain insufficient alone.

Exit condition: an attacker cannot gain Trusted Network behavior merely by reproducing local addressing or SSID characteristics.

### P0-13 - Administrative authority and operator surfaces

Translate the DNP-style Authority Grant model into Drawbridge administration.

Define:

- Site/Tenant admin boundaries;
- enrollment/recovery authority;
- Revocation Operator;
- policy author/test/promote;
- Records reader/export/retention;
- system/storage administration;
- delegation;
- separation-of-duties policy;
- admin MFA/network restrictions;
- exercise Records.

Exit condition: role names remain convenience templates, not hidden authorization shortcuts.

### P0-14 - Break-glass contract

Define emergency recovery without creating a vendor backdoor.

Resolve:

- who may invoke;
- independent authentication;
- exact scope;
- duration/expiry;
- allowed operations;
- prohibited bypasses;
- audit/Records;
- post-event review;
- behavior during Controller/domain/PKI incidents.

Exit condition: break-glass is explicit, bounded, attributable, and testable.

### P0-15 - Failure-mode matrix and availability thresholds

Convert the threat-model principles into per-dependency behavior.

Cover at least:

- Front Distributor, including complete primary-ingress loss under each configured Gateway Placement Profile behavior;
- Gateway;
- Controller;
- AD;
- NPS/MFA;
- PKI validation;
- DNS;
- Records;
- DRS;
- enterprise firewall/handoff;
- transport loss;
- disk/spool pressure;
- reconnect storms.

Define which operations continue, suspend, fail closed, or enter Limited/Degraded state.

Exit condition: no implementation chooses fail-open/fail-closed behavior ad hoc.

### P0-16 - Narrow first pilot and acceptance matrix

Freeze the first serious deployment boundary.

Define:

- Device class/Windows versions;
- Domain-Managed versus Shared-Service scope for first pilot;
- number/topology of Front Distributors/Gateways;
- Gateway Placement Profile(s), including Domain/Shared-Service separation and pilot fallback behavior;
- Resource types;
- AD/PKI/NPS integrations;
- routing model;
- DNS mode;
- HA tests;
- PD Ops/911/EMO/network/systems/security/application-owner acceptance;
- explicit out-of-scope features.

Exit condition: Phase 1-9 engineering has one concrete target rather than every possible deployment.

### P0-17 - Repository governance before production code

Before serious implementation:

- branch/ruleset protection;
- required review/checks;
- CI baseline;
- SECURITY.md;
- CODEOWNERS/ownership model if appropriate;
- release/signing workflow;
- contribution rules;
- ADR location/convention;
- issue/milestone mapping from this Phase 0 review.

Exit condition: code cannot bypass the engineering/change-control rules the product itself requires.

## Phase 0 exit criteria

Phase 0 is complete when:

1. P0-04 through P0-17 are closed or explicitly deferred with a documented reason;
2. architecture/terminology/threat/routing/Records/HA documents agree;
3. transport and HA recovery ADRs are sufficient for a Phase 1 prototype;
4. identity/authorization/PKI contracts no longer depend on unspecified magic;
5. Records v0 and time-ordering semantics exist before implementation fragments the event model;
6. Windows Agent privilege boundaries are frozen before privileged code is written;
7. failure behavior is deliberate rather than implementation-defined;
8. the first pilot has a narrow, testable operational definition;
9. repository governance is in place before serious production code.

## Immediate next step

Proceed with **P0-04 - User authorization assertion and User Session binding**.

P0-03 now establishes that the Gateway cannot accept an Agent statement such as “this User is authorized” as authority by itself. P0-04 must define the bounded assertion that turns successful Windows/AD/NPS/RADIUS/federated authentication into a current Drawbridge User Session without sending or replaying reusable User credentials.
