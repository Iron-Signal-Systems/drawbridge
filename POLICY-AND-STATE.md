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
- cached policy expired;
- Gateway available but directory unavailable;
- CRL/OCSP temporarily unreachable;
- trusted-network proof unavailable;
- Records backend unavailable;
- Device clock skew.

No hidden implicit fail-open behavior.

A declared PKI compromise is fail-closed for all new trust establishment through the affected issuer.

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
