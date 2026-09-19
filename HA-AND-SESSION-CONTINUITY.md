# Drawbridge High Availability and Session Continuity

## 1. Purpose

This document defines the Phase 0 high-availability architecture contract for Drawbridge ingress, Gateway selection, component failure, and logical session continuity.

It intentionally separates architecture invariants from implementation mechanisms that still require transport and HA prototyping.

## 2. Stable Drawbridge service ingress

Production Agents normally target a stable Drawbridge Service Address rather than a named Gateway. The effective Gateway Placement Profile defines the only authorized exception when the configured Front Distributor tier is unavailable.

Conceptually:

```text
Drawbridge Agent
    |
    v
Drawbridge Service Address
    |
    v
Drawbridge Front Distributor tier
    |
    v
eligible Drawbridge Gateway
```

The Service Address may eventually be implemented with a VIP, FQDN plus redundant ingress, CARP/VRRP-style ownership, anycast, cloud ingress, or another appropriate mechanism.

The implementation mechanism is not the architectural identity.

DNS or possession of the Service Address is not itself proof of Drawbridge peer trust. Normal cryptographic identity and authorization requirements remain in force.

## 3. Drawbridge Front Distributor

The Drawbridge Front Distributor is a narrow ingress traffic-placement component.

Its responsibilities are limited to functions such as:

- accept/route supported Drawbridge transport;
- identify eligible Gateway targets;
- distribute new transport connections/sessions;
- preserve affinity where the transport/session design requires it;
- remove failed, not-ready, or draining Gateways from new placement;
- expose ingress and Gateway-placement health;
- record placement/failover decisions.

It is not a Controller and does not own:

- Device authorization;
- User authorization;
- Tenant/Resource authorization;
- policy compilation;
- directory administration;
- PKI administration;
- endpoint administration;
- Gateway OS administration;
- Records administration.

A placement decision means only:

> send this Drawbridge transport to this eligible Gateway.

The selected Gateway must independently validate the Device/session and enforce the Drawbridge authorization state for which it is responsible.

## 4. Front Distributor availability

The Front Distributor tier is redundant in production.

A Front Distributor node is intended to be disposable. Its failure must not redefine Device identity, User authorization, or the logical Device Session.

Preferred state characteristics:

- configuration is versioned/reconstructable;
- Gateway health is rediscoverable;
- routing/placement behavior is deterministic;
- precious per-session state is minimized;
- node replacement does not require endpoint reconfiguration.

The exact production topology remains an implementation choice. A small on-premises deployment may use a redundant pair; larger or geographically resilient deployments may have additional ingress failure domains.

## 5. Gateway Placement Profile and total-ingress failure

Drawbridge does not invent a failure route when ingress is unavailable.

Each in-scope Device population is assigned a versioned **Gateway Placement Profile** defining:

- primary Drawbridge Service Address;
- authorized Front Distributor/ingress set;
- eligible Gateway pool;
- total Front Distributor failure behavior;
- optional secondary ingress;
- optional direct-Gateway fallback targets;
- deterministic fallback target order/selection behavior.

Domain-Managed and Shared-Service environments may intentionally use different profiles and network domains.

Example:

```text
COUNTY-DOMAIN
  primary: db-domain.county.gov
  ingress: DB-DOM-FP-01 / DB-DOM-FP-02
  gateways: DB-DOM-GW-01 / DB-DOM-GW-02 / DB-DOM-GW-03

REGIONAL-SHARED
  primary: db-shared.county.gov
  ingress: DB-SH-FP-01 / DB-SH-FP-02
  gateways: DB-SH-GW-01 / DB-SH-GW-02 / DB-SH-GW-03
```

Failure of the Domain ingress does not authorize a Domain Agent to select a Shared-Service Gateway, and the reverse is also true.

Supported total Front Distributor failure policies are:

### FAIL_CLOSED

The Agent does not establish a new alternate Drawbridge ingress path. Existing behavior follows the defined session/transport failure contract.

### SECONDARY_INGRESS

The Agent uses a separately configured equivalent Drawbridge ingress failure domain named by the Placement Profile.

The secondary path preserves the normal architecture:

```text
Agent
  ->
Drawbridge Service Address / ingress
  ->
Front Distributor
  ->
Gateway
```

### DIRECT_GATEWAY_FALLBACK

The Agent may connect directly only to Gateway targets explicitly named by the Placement Profile.

Direct fallback is not an improvised bypass. Every eligible direct-fallback Gateway must be intentionally configured, exposed, cryptographically identified, firewalled, health-checked, and tested for that role before production use.

The Placement Profile defines deterministic target selection/order. The Agent must not randomly choose an arbitrary reachable Gateway.

Direct fallback changes only the transport destination. It does not change:

- Device identity;
- User identity/authorization;
- Tenant;
- Resource policy;
- revocation state;
- Gateway peer authentication;
- Gateway independent session/authorization enforcement.

Direct-fallback mode is a degraded operational state and must be visible and recorded.

### Profile availability

The Agent must already possess the signed/versioned effective Gateway Placement Profile needed for failure handling. Total-ingress recovery must not depend on receiving new routing instructions from the Controller after ingress has failed.

## 6. Gateway pool and maintenance

The Front Distributor selects only Gateways currently eligible for new traffic.

Gateway eligibility must eventually consider more than a listening port, including at least:

- transport listener health;
- control-channel health;
- current configuration/policy generation;
- certificate/trust state;
- virtual-address/session capacity;
- enterprise handoff health;
- Records/spool pressure within safe bounds;
- explicit maintenance/drain state.

Maintenance should support draining a Gateway:

```text
GW-01  READY
GW-02  DRAINING
GW-03  READY

new placement:
  GW-01 / GW-03

existing GW-02 sessions:
  finish, migrate, or resume according to the session contract
```

## 7. Session continuity invariant

Failure of a Transport Path, Front Distributor, or Gateway does not by itself redefine:

- Device identity;
- User identity;
- current authorization;
- logical Device Session;
- virtual Device identity/address.

Those facts may change only for their own defined security/lifecycle reasons.

The transport connection used to carry a Device Session is not the Device Session itself.

Likewise:

```text
Transport connection != Device Session
Front Distributor assignment != Device identity
Gateway assignment != Device identity
Gateway assignment != User authorization
```

## 8. Front Distributor failure

If one Front Distributor fails and another member of the same ingress service remains healthy:

1. the stable Service Address remains/re-becomes reachable through the surviving ingress node;
2. the Agent may experience a bounded transport interruption;
3. transport resumes or re-establishes through the same Drawbridge service;
4. the logical Device Session is preserved where its current security state remains valid;
5. the Gateway may remain the same or change according to transport/session recovery behavior.

Endpoint configuration does not change.

If all Front Distributors defined for the primary ingress fail:

1. the Agent identifies its current signed/versioned Gateway Placement Profile;
2. it follows that profile's configured behavior only;
3. FAIL_CLOSED creates no alternate ingress;
4. SECONDARY_INGRESS uses only the configured secondary Drawbridge ingress;
5. DIRECT_GATEWAY_FALLBACK uses only explicitly authorized fallback Gateways and the configured deterministic target order/selection policy;
6. no failure condition permits crossing into a different Domain/Shared-Service Placement Profile merely because another path is reachable;
7. normal cryptographic identity, revocation, session, and Resource authorization remain in force;
8. degraded/fallback state and recovery are recorded.

## 9. Gateway failure

If a Gateway fails:

1. the Front Distributor stops assigning new traffic to that Gateway;
2. the Agent reconnects/resumes through the stable Service Address;
3. an eligible Gateway is selected;
4. the new Gateway independently validates resume/session authority;
5. the logical Device Session continues where current security state permits.

Gateway failure is not Device revocation and does not require Device re-enrollment.

## 10. Controller and Records independence

Front Distributor/Gateway data-plane continuity does not make the Controller or Records part of each forwarded packet.

Temporary Controller, Records, or DRS unavailability follows the existing threat/failure contracts.

The Front Distributor does not gain authorization authority merely because the Controller is unavailable.

## 11. Session-state implementation remains open

Phase 0 does not yet choose between:

- replicated/shared Gateway session state;
- portable cryptographically protected resume state/token;
- a hybrid model.

The selected model must satisfy:

- replay resistance;
- current revocation enforcement;
- Device/User/Tenant binding;
- policy-generation binding;
- bounded resume lifetime;
- virtual identity/address continuity rules;
- Gateway failover;
- key rotation;
- failure/recovery Records.

The Front Distributor should not become the only irreplaceable owner of active session state.

## 12. Transport-aware distribution remains open

QUIC is a strong transport candidate because its connection model can survive lower-layer address/port changes.

If QUIC is selected, Connection-ID-aware routing may reduce Front Distributor state and improve mobility/failover.

Phase 0 does not adopt a specific QUIC load-balancing draft or proprietary encoding as a standard.

The Drawbridge logical Device Session remains distinct from any specific QUIC connection or Connection ID.

## 13. Records requirements

HA/distribution events are first-class Records.

Useful facts include:

- Gateway Placement Profile identity/version;
- configured total-ingress-failure policy;
- Service Address/ingress identity;
- Front Distributor identity;
- selected Gateway;
- fallback target/selection reason where applicable;
- previous Gateway where applicable;
- placement reason;
- Gateway eligibility/health transition;
- Front Distributor health transition;
- drain start/completion;
- transport interruption;
- resume attempt/result;
- logical Device Session preserved or replaced;
- virtual identity/address preserved or changed;
- data-plane restoration time.

Records must make it possible to reconstruct an outage/failover timeline.

## 14. Acceptance testing

HA testing must deliberately inject failures.

At minimum, later implementation acceptance should include:

- kill active/serving Front Distributor;
- remove/restore Service Address ownership or equivalent ingress;
- fail the entire primary Front Distributor tier and verify the configured Gateway Placement Profile behavior;
- verify Domain-Managed and Shared-Service profiles cannot cross-select each other's ingress/Gateway pools during failure;
- where direct fallback is enabled, verify only explicitly configured fallback Gateways are attempted in the configured deterministic order/selection behavior;
- kill active Gateway;
- drain Gateway for maintenance;
- restore Gateway and prove eligibility checks before new placement;
- Controller unavailable while established traffic continues according to policy;
- Records unavailable while bounded spooling operates;
- transport-path change during/near Gateway or ingress failure;
- reconnect storm after common outage.

For an ingress failure test, verify:

- Service Address remains/re-becomes reachable;
- Agents resume/re-establish;
- Device identity is unchanged;
- current User authorization does not silently broaden;
- virtual identity/address follows the defined continuity contract;
- no unauthorized Resource access appears;
- Records correlate the failure and recovery.

Exact interruption SLOs/targets are not frozen in Phase 0.

## 15. Phase 0 locked versus open

Locked architecture:

- stable Drawbridge Service Address above replaceable Gateways;
- redundant Front Distributor tier in production;
- Front Distributor authority is traffic placement only;
- Gateway independently validates/enforces Drawbridge authorization;
- Front Distributor state should be disposable/reconstructable where practical;
- Gateway Placement Profile is the authoritative configuration object for primary ingress/Gateway placement and total Front Distributor failure behavior;
- total-ingress behavior is explicit: FAIL_CLOSED, SECONDARY_INGRESS, or DIRECT_GATEWAY_FALLBACK;
- direct fallback may target only explicitly prepared/named Gateways and uses deterministic configured selection behavior;
- failure never authorizes crossing between Domain-Managed and Shared-Service Placement Profiles;
- fallback changes transport placement only and does not weaken Gateway/session/Resource authorization;
- the Agent has the signed/versioned effective Placement Profile before a failure requires it;
- Transport Path/Front Distributor/Gateway failure does not by itself redefine Device/User identity or the logical Device Session;
- HA/failover/fallback events are recorded and intentionally tested.

Still open for prototype/ADR work:

- active/standby versus active/active Front Distributors;
- on-prem Service Address mechanism such as CARP/VRRP/anycast/external load balancer;
- exact transport protocol;
- transparent versus transport-aware Front Distributor mechanics;
- exact Gateway health/eligibility contract;
- replicated session state versus cryptographic resume state/token versus hybrid;
- Connection-ID-aware routing if QUIC is selected;
- exact representation/distribution/cache contract for Gateway Placement Profiles;
- direct-fallback Gateway exposure/health mechanism and deterministic selection algorithm;
- geographic ingress/site affinity and DR behavior;
- exact interruption/session-resume timing targets.
