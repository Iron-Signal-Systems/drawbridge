# Policy and State Model

## 1. Policy model

Drawbridge should use a small set of understandable objects:

- Devices
- Users / Identity mappings
- Tenants / Agencies
- Resources
- Networks
- Access Profiles
- Remote Management Policies
- Gateways
- Gateway Placement Profiles
- Sites

The product should not create deep, opaque override hierarchies.

## 2. Resource objects

Network administrators should be able to model meaningful resources such as:

```text
CAD
RMS
GIS
County DNS
Domain Services
Endpoint Management
```

Each resource expands deterministically to actual network definitions:

- IP/subnet;
- FQDN/domain where supported;
- protocol;
- port;
- direction;
- optional application/process constraints.

The underlying L3/L4 rules must remain inspectable.

## 3. Directionality

Inbound and outbound authorization are separate.

A device being allowed to reach a service does not imply that the service or IT can initiate connections back to the device.

Shared-service device default:

```text
outbound: explicitly allowed resources only
inbound: none
```

## 4. Orthogonal authoritative state domains

Drawbridge does not use one mixed state enum.

DeviceSessionState: NONE, CREATING, ACTIVE, SUSPENDED, TERMINATING, TERMINATED.

TransportState: OFFLINE, CONNECTING, ESTABLISHED, SUSPENDED, RESUMING, FAILED.

DeviceAuthorizationState: UNKNOWN, AUTHENTICATING, AUTHORIZED, DENIED, REVOKED.

UserAuthorizationState: NO_USER, AUTHENTICATING, AUTHORIZED, DENIED, STALE.

NetworkTrustState: UNKNOWN, UNTRUSTED, TRUSTED.

PolicyPosture: NORMAL, LIMITED, QUARANTINED, BLOCKED.

HealthState: HEALTHY, DEGRADED, FAILED.

State, reason, and event are different concepts.

Exact allowed transitions within and across the authoritative state domains must be formalized before implementation.

## 5. Derived user-facing states

The normal endpoint UI should remain simple.

These are derived UI views, not authoritative security states:

- Connected
- Device Only
- Trusted Network
- Limited
- Disconnected

Detailed diagnostics are for IT.

## 6. Device plane versus user plane

Example:

```text
DEVICE_CONNECTED
  device management allowed
  domain bootstrap allowed
  user business resources denied

USER_AUTHORIZED
  device plane remains
  user resource policy added
```

User logout/disconnect does not inherently destroy device-management connectivity.

## 7. Network classification

Minimum classes:

- Trusted Enterprise
- Untrusted
- Unknown
- Quarantine/Restricted (if needed)

Trust classification must be explainable and recorded.

## 8. Policy decision explanation

Every allow/deny/tunnel/direct decision should expose:

- subject/device;
- user;
- tenant;
- current network state;
- resource;
- matched rule/profile;
- policy version;
- final action;
- human-readable reason.

Example:

```text
Decision: DENY

Device: MDT-027
User: AGENCY\jsmith
Destination: RMS-DB01:1433
Tenant: Village PD

Reason:
  device authorized: YES
  user authorized: YES
  resource exists: YES
  tenant access to RMS-DB: NO

Policy version:
  DB-POL-000184
```

## 9. Policy conflict handling

Drawbridge should prefer either:

- deterministic restrictive intersection; or
- explicit configuration rejection when policy intent is ambiguous.

It should not silently depend on obscure rule order.

For tunneled enterprise traffic, effective access is the restrictive intersection of Agent local permission/path selection, Gateway independent authorization, and enterprise firewall/network acceptance. `TUNNEL` is not an allow decision. A broader grant at one enforcement point never overrides a restriction at another.

Temporary Agent/Gateway policy-generation skew must not create a transient broader grant: new access is usable only when every required enforcement point has a compatible permit, while a deny/removal at either point is sufficient to stop matching traffic there. See [ENFORCEMENT-BOUNDARIES.md](ENFORCEMENT-BOUNDARIES.md).

## 10. Split routing

Base functionality should include:

- IP/subnet routing;
- FQDN/domain-aware routing;
- direct versus tunnel policy;
- DNS policy;
- trusted-network bypass;
- future application-aware routing.

These must not be separately licensed feature packs.

## 11. Administrative authority evaluation

Security-sensitive administrative operations evaluate a current scoped Authority Grant against exact actor, Site/Tenant, operation, protected target, governed scope, time, and policy. Revocation authority may remove trust without creating new trust.

## 12. Fail-open/fail-closed

Security failure behavior must be explicit and administrator-configurable within safe bounds.

Examples requiring defined behavior:

- Controller unreachable;
- all Front Distributors for the primary ingress unavailable;
- cached policy expired;
- Gateway available but directory unavailable;
- CRL/OCSP temporarily unreachable;
- trusted-network proof unavailable;
- Records backend unavailable;
- Device clock skew.

No hidden implicit fail-open behavior.

Total Front Distributor failure is governed by the Device's current signed/versioned Gateway Placement Profile. The only permitted profile outcomes are explicit failure behavior such as FAIL_CLOSED, SECONDARY_INGRESS, or DIRECT_GATEWAY_FALLBACK to pre-authorized targets. Availability failure does not create new security authority.

Unavailable and compromised dependencies are different states. Loss of a non-forwarding dependency does not automatically invalidate independently current established authorization, while a fresh decision must not invent authorization when a required authority is unavailable.

A declared PKI compromise is fail-closed for all new trust establishment through the affected issuer once that compromise state is known to the enforcement point.

## 13. DNS policy

DNS behavior is explicit policy.

Supported policy outcomes include:

- use local connection resolver;
- use Drawbridge/enterprise resolver for protected namespaces only;
- force all DNS through Drawbridge;
- use hybrid resolver selection by namespace/resource.

DNS policy is evaluated before FQDN-based route authorization.

A protected namespace must not be silently resolved by an untrusted local resolver.

Unauthorized alternate DNS mechanisms such as hard-coded resolvers, DoH, or DoT may be blocked or redirected according to site policy.
