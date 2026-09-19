# Routing and Deployment Models

## 1. Principle

Drawbridge must not force customers to replace their existing routing or firewall design.

Routing is modeled on two independent axes:

1. **Endpoint traffic selection** — what traffic the Agent sends into Drawbridge.
2. **Enterprise handoff** — how the Drawbridge Gateway delivers authorized traffic to the destination environment.

This allows a simple DMZ deployment to remain simple while still supporting more complex proxy/connector deployments where required.

Production ingress is a separate HA layer: the Agent normally targets the stable Drawbridge Service Address, the redundant Front Distributor tier selects an eligible Gateway, and the Gateway performs Drawbridge authorization/enforcement before enterprise handoff. The Device's Gateway Placement Profile defines the only authorized behavior if the configured Front Distributor tier is unavailable.

## 1.1 Gateway Placement Profiles and ingress failure

Gateway Placement Profiles keep ingress/failure behavior explicit and aligned with the customer's network design.

A profile binds a Device population or trust/deployment domain to:

- its primary Drawbridge Service Address;
- its authorized Front Distributor/ingress set;
- its eligible Gateway pool;
- its total Front Distributor failure policy;
- any secondary ingress;
- any direct-Gateway fallback targets and deterministic selection/order.

Domain-Managed and Shared-Service paths may be intentionally separate.

Example:

```text
COUNTY-DOMAIN
  Service Address: db-domain.county.gov
  Front Distributor set: DB-DOM-FP-*
  Gateway pool: DB-DOM-GW-*
  total-ingress failure: SECONDARY_INGRESS

REGIONAL-SHARED
  Service Address: db-shared.county.gov
  Front Distributor set: DB-SH-FP-*
  Gateway pool: DB-SH-GW-*
  total-ingress failure: DIRECT_GATEWAY_FALLBACK
  fallback targets: DB-SH-GW-01, then DB-SH-GW-03
```

Drawbridge must not respond to a Domain ingress failure by selecting a Shared-Service Gateway, or vice versa.

Where DIRECT_GATEWAY_FALLBACK is enabled, those Gateways are deliberate emergency ingress points with explicit exposure, identity, firewall, health, and test requirements. Direct fallback changes only transport placement; normal Gateway authorization remains mandatory.

## 2. Endpoint traffic selection

### 2.1 Selective / split tunnel

Only configured enterprise resources enter Drawbridge.

Everything else uses the endpoint's normal local Internet connection.

Example:

```text
*.xyz.local.domain   -> DRAWBRIDGE
10.40.20.0/24        -> DRAWBRIDGE
everything else      -> DIRECT
```

This should be normal base functionality.

A customer that only needs internal traffic for `xyz.local.domain` should not have to build a full-tunnel or proxy architecture.

### 2.2 Full tunnel

All normal endpoint traffic enters Drawbridge.

The enterprise firewall may then decide:

- internal routing;
- Internet egress;
- filtering;
- inspection;
- logging.

This is useful where the organization requires all remote traffic to traverse the enterprise security stack.

### 2.3 Trusted-network bypass

On a verified trusted enterprise network, Drawbridge may suspend or bypass the remote overlay according to policy.

The transition must preserve session continuity where technically possible.

## 3. Enterprise handoff

### 3.1 Routed DMZ handoff — preferred simple model

This is the preferred default for organizations that already trust and understand their firewall.

```text
Remote endpoint
      |
      | Drawbridge mobility transport
      v
Drawbridge Service Address
      |
      v
Redundant Front Distributor tier
      |
      v
+-------------------+
| Drawbridge Gateway |
|       DMZ          |
+---------+----------+
          |
          | routed virtual Device traffic
          v
+-------------------+
| Existing Firewall |
+---------+---------+
          |
          v
   Internal networks
```

Drawbridge terminates the mobility session and exposes an authorized virtual Device address/pool to the existing firewall.

The enterprise firewall remains authoritative for:

- inter-zone routing;
- internal segmentation;
- NAT where desired;
- conventional security policy;
- Internet egress in full-tunnel mode.

Drawbridge remains authoritative for:

- stable Drawbridge service ingress, Gateway Placement Profiles, and Gateway placement;
- device/session identity;
- user authorization;
- tunnel eligibility;
- Agent-side traffic selection;
- Drawbridge-specific resource policy;
- mobility/session continuity;
- Records.

These responsibilities are split across enforcement boundaries: the Agent selects DIRECT/TUNNEL/DENY locally; the Front Distributor performs placement only; the Gateway independently authorizes tunneled enterprise traffic; and the enterprise firewall remains an additional independent deny/routing boundary. See [ENFORCEMENT-BOUNDARIES.md](ENFORCEMENT-BOUNDARIES.md).

### 3.2 Return routing

Preferred behavior is preservation of the Drawbridge virtual Device address.

Example:

```text
Drawbridge virtual pool:
10.250.0.0/16

Firewall route:
10.250.0.0/16 -> Drawbridge Gateway inside/transit address
```

Return traffic follows:

```text
internal host
   ->
enterprise firewall
   ->
Drawbridge virtual pool route
   ->
Drawbridge Gateway
   ->
logical Device Session
```

Static routing is sufficient for small environments.

Future support may include dynamic route advertisement where justified.

### 3.3 NAT handoff

Drawbridge may optionally source-NAT traffic when a customer cannot or will not route the Drawbridge virtual pool.

This is a compatibility option, not the preferred model.

Tradeoff:

```text
routed handoff:
firewall can see per-Device virtual source addresses

NAT handoff:
firewall may only see Drawbridge Gateway/proxy source addresses
```

Drawbridge Records must retain the actual endpoint/device/user identity regardless.

### 3.4 Service proxy / connector handoff

Some environments cannot expose routed virtual Device address pools into the host network, have overlapping address spaces, or require highly constrained shared-service access.

For those cases Drawbridge may support an internal service proxy/connector.

Conceptually:

```text
Remote Device
    |
    v
Drawbridge Gateway (DMZ)
    |
    | authenticated Drawbridge service channel
    v
Drawbridge Service Connector
    |   inside host network
    |
    +--> CAD
    +--> RMS
    +--> GIS
```

The internal connector may establish new service-side connections instead of requiring the enterprise to route the remote virtual Device address pool.

The Service Connector is a narrowly scoped Resource bridge, not a general internal gateway or jump host. It may reach only explicitly assigned Resources/supporting services, and enterprise firewall policy should independently enforce the same boundary.

Connector service identities are host/function scoped and provide no general domain or endpoint administrative authority.

This is useful for:

- shared-services environments;
- overlapping RFC1918 networks;
- organizations that do not permit routed remote Device address pools;
- application/service-specific exposure;
- strong tenant isolation.

Proxy/connector mode must remain optional. Customers should not be forced into this complexity when normal routing is sufficient.

## 4. Domain/FQDN selective routing

A common deployment should be easy:

> Send only traffic for `xyz.local.domain` through Drawbridge. Send everything else directly.

A resource may therefore be defined using a private DNS namespace.

Example:

```text
Resource:
  Internal-XYZ

DNS namespace:
  *.xyz.local.domain

Resolver:
  enterprise/private resolver

Action:
  tunnel

Default:
  direct
```

Drawbridge must preserve DNS context and bind FQDN policy to observed resolution-at-time.

Important requirements:

- queries for protected private namespaces may need to use enterprise/private DNS;
- Drawbridge Records preserve the exact DNS answer observed at the time;
- CNAME chains must be handled deterministically;
- address bindings must respect TTL/refresh semantics;
- stale addresses must age out;
- DNS rebinding and hostile/local resolver behavior must be considered;
- wildcard/domain rules must not become an uncontrolled broad-IP bypass;
- direct Internet DNS should remain available for non-protected namespaces according to policy.

The exact Windows DNS implementation is not yet frozen.

## 5. Example: simple county deployment

Requirement:

- stable Drawbridge Service Address;
- redundant Front Distributor tier;
- Drawbridge Gateway pool in DMZ;
- existing firewall remains the router/firewall;
- only county-private applications use Drawbridge;
- normal Internet stays local.

```text
Endpoint
  |
  +-- cad.xyz.local.domain ----> Drawbridge
  +-- rms.xyz.local.domain ----> Drawbridge
  +-- 10.40.20.0/24 -----------> Drawbridge
  |
  +-- public Internet ----------> local/direct
```

Enterprise:

```text
Drawbridge Service Address
    |
Front Distributor
    |
Drawbridge Gateway
    |
    | source = 10.250.x.x virtual Device addresses
    v
Existing Firewall
    |
    +--> CAD zone
    +--> RMS zone
    +--> DNS
```

The firewall can continue using the same network-admin workflow it already uses.

## 6. Example: full-tunnel deployment

```text
Endpoint
   |
   | all traffic
   v
Drawbridge Gateway
   |
   v
Enterprise Firewall
   |
   +--> internal resources
   |
   +--> Internet egress/security stack
```

Drawbridge does not need to duplicate the firewall's entire security stack.

## 7. Example: shared-services proxy deployment

Agency A uses a service hosted by County B.

No domain trust and no general routed network relationship exists.

```text
Agency A device
      |
      | Drawbridge
      v
DMZ Gateway
      |
      v
Internal Service Connector
      |
      +--> CAD:443
      +--> RMS:443
```

Policy may be:

```text
Agency A
  CAD HTTPS -> ALLOW
  RMS HTTPS -> ALLOW
  SMB       -> DENY
  RDP       -> DENY
  other     -> DENY
```

The connector does not imply host administrative rights on Agency A's endpoint.

## 8. Records requirements

Routing decisions are first-class Records.

A connection record should be able to state:

```text
endpoint_selection:
  TUNNEL

reason:
  destination matched *.xyz.local.domain

enterprise_handoff:
  ROUTED_DMZ

gateway:
  DB-GW-01

virtual_source:
  10.250.14.37

policy_version:
  DB-POL-000184
```

For proxy mode:

```text
enterprise_handoff:
  SERVICE_PROXY

connector:
  DB-CONN-02

backend:
  CAD01:443
```

## 9. Security boundaries

Transport networks are assumed potentially hostile. IP addressing, DHCP, DNS, SSID, subnet, gateway address, and apparent enterprise topology do not prove trust. DNS locates an endpoint; cryptographic peer authentication proves the Drawbridge peer.

Drawbridge must not interpret "firewall-owned routing" as "Drawbridge allows everything."

The minimum security boundary remains:

```text
authenticated device
AND
authorized user when required
AND
authorized resource
AND
current network state
```

The existing firewall may apply additional restrictions.

This creates defense in depth without forcing duplicated policy everywhere.

Gateway Placement Profile selection and Front Distributor placement are not authorization. Gateway hosts independently validate/enforce Drawbridge session and Resource policy. Front Distributor and Gateway identities are separately scoped. A fallback path may change where transport arrives, but never what the Device/User/Tenant is authorized to reach.

Gateway hosts use host-specific service identities and receive no general domain administrative authority. A compromised Device may generate malicious traffic, so authenticated Device traffic remains constrained by Drawbridge Resource policy and the enterprise firewall.

For FQDN Resources, an Agent-observed DNS answer may drive local tunnel selection and Records but cannot by itself expand Gateway authorization. The Gateway requires the current authoritative/verifiable FQDN binding defined by the DNS architecture.

## 10. Recommended v1 priority

Recommended implementation order:

1. selective/split tunnel;
2. full tunnel;
3. routed DMZ handoff;
4. optional NAT compatibility;
5. trusted-network bypass;
6. service proxy/connector mode after the routed model is stable.

The simplest deployment should remain the easiest and most thoroughly tested.

## 11. DNS-selected routing

FQDN-based split routing depends on Drawbridge DNS policy.

Example:

```text
*.xyz.local.domain
    DNS -> enterprise resolver via Drawbridge
    traffic -> Drawbridge

everything else
    DNS -> local connection resolver
    traffic -> direct
```

Another site may require:

```text
all DNS -> Drawbridge filtering resolver
internal traffic -> Drawbridge
public traffic -> direct
```

DNS path and traffic path are therefore related but independently configurable.

Drawbridge must record both decisions.
