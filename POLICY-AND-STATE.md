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

## 4. Session states

Initial state model:

- `OFFLINE`
- `DEVICE_AUTHENTICATING`
- `DEVICE_CONNECTED`
- `USER_AUTHENTICATING`
- `USER_AUTHORIZED`
- `TRUSTED_NETWORK`
- `SESSION_SUSPENDED`
- `SESSION_RESUMING`
- `LIMITED`
- `QUARANTINED`
- `REVOKED`
- `DEGRADED`

Exact allowed transitions must be formalized before implementation.

## 5. User-facing states

The normal endpoint UI should remain simple.

Suggested user-visible states:

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

## 11. Fail-open/fail-closed

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

## 12. DNS policy

DNS behavior is explicit policy.

Supported policy outcomes include:

- use local connection resolver;
- use Drawbridge/enterprise resolver for protected namespaces only;
- force all DNS through Drawbridge;
- use hybrid resolver selection by namespace/resource.

DNS policy is evaluated before FQDN-based route authorization.

A protected namespace must not be silently resolved by an untrusted local resolver.

Unauthorized alternate DNS mechanisms such as hard-coded resolvers, DoH, or DoT may be blocked or redirected according to site policy.
