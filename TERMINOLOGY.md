# Drawbridge Terminology

## 1. Purpose

This document defines the canonical names for Drawbridge actors, components,
sessions, and core architecture concepts.

These terms are normative across Drawbridge documentation, implementation,
Records, APIs, diagnostics, tests, and operator interfaces.

When identity, authority, policy, or trust matters, use the exact term rather
than a generic synonym.

## 2. Core actors and components

### User

A human identity operating through a Device.

Examples include:

- Active Directory user;
- local Windows user;
- federated user;
- approved shared-services user.

A User is not the Device and is not the Drawbridge Agent.

### Device

The endpoint hardware and operating-system instance participating in Drawbridge.

For the initial Windows/public-safety target, a Device is typically a
domain-managed Windows computer.

A Device may have an authenticated Device Session when no User is logged on.

### MDT

A Device deployed specifically as a mobile data terminal.

MDT is a deployment role of a Device, not a different trust primitive.

### Drawbridge Agent

The Drawbridge software installed on and operating from a Device.

Drawbridge defines two explicit Agent implementations/contracts:

- Domain Agent — hosting-organization/domain-managed Device;
- Shared-Service Agent — Device administered by a participating agency or other independent authority.

They may share narrow libraries but remain distinct top-level implementations.

The Agent is responsible for endpoint-side Drawbridge functions such as:

- Device authentication;
- transport establishment;
- virtual networking;
- local traffic classification and enforcement;
- trusted-network detection;
- DNS policy functions;
- local Records collection;
- health reporting.

The Agent is software. It is not the User and it is not the Device itself.

### Drawbridge Service Address

The stable production ingress target used by Drawbridge Agents to reach the Drawbridge service.

It may be represented by an FQDN plus a deployment-specific listener/VIP/ingress mechanism. Agents target the Drawbridge Service Address rather than a named Gateway.

The Service Address is discovery/ingress state, not proof of peer trust.

### Drawbridge Front Distributor

The narrow ingress component that places Drawbridge transport onto an eligible Drawbridge Gateway.

It may evaluate Gateway health, capacity, drain state, and transport affinity. It does not grant Device/User/Tenant/Resource authorization and is not a Controller.

Production uses a redundant Front Distributor tier. Individual Front Distributor nodes should be disposable/reconstructable.

### Drawbridge Gateway

The server-side Drawbridge enforcement/forwarding component that receives Drawbridge transport through the production ingress tier and terminates/validates the Gateway-side session.

The Gateway is responsible for functions such as:

- Device and session authentication;
- virtual-address binding;
- Drawbridge policy enforcement;
- packet forwarding;
- session suspend/resume support;
- path migration support;
- Gateway-side Records;
- high-availability participation.

The Gateway does not replace the enterprise firewall.

### Drawbridge Controller

The management and control component responsible for functions such as:

- configuration;
- policy compilation;
- directory synchronization;
- User and Device entitlement;
- PKI enrollment workflow;
- Site and Tenant configuration;
- test/staging orchestration;
- version history;
- promotion and rollback;
- Records indexing/search;
- health and readiness status.

The Controller must not be required for every forwarded packet.

### Drawbridge Recovery Store

An independent configuration-recovery subsystem outside the production Windows/AD trust boundary.

It receives complete Gateway/Controller recovery objects and preserves canonical objects that are immutable from birth.

It is a recovery anchor, not a live policy authority.

### Service Connector

An optional internal Drawbridge component used where direct routed handoff from
a Gateway is inappropriate.

Typical uses include:

- constrained shared-services access;
- overlapping private address space;
- environments that do not permit routed remote Device address pools;
- service-specific connectivity across a security boundary.

The Service Connector does not imply domain trust or endpoint administrative
authority.

## 3. Sessions and paths

### Device Session

A logical Drawbridge connectivity session associated with an authenticated
Device.

A Device Session may exist before User logon and may provide only explicitly
authorized Device/bootstrap connectivity.

### User Session

User authorization associated with a User operating through an existing Device
Session.

A User Session may add explicitly authorized business resources without
replacing the Device identity or Device Session.

User logout does not inherently destroy the Device Session.

### Transport Path

The current underlying network path used by the Agent to reach a Gateway.

Examples include:

- Wi-Fi;
- LTE/5G;
- Ethernet;
- another routed Internet connection.

A Transport Path may change while Device identity and the logical Device Session
remain stable.

## 4. Policy and deployment terms

### Resource

A service or network destination represented in Drawbridge policy.

Examples include:

- CAD;
- RMS;
- GIS;
- Domain Services;
- enterprise DNS;
- endpoint-management services;
- an IP/subnet;
- an FQDN/domain plus protocol and port.

### Site

A Drawbridge deployment boundary containing the production environment and its
normal associated HA, DR, and test/staging capacity.

Site is also the intended primary commercial licensing unit.

### Tenant / Agency

An administrative and policy boundary representing an agency or other
participating organization within a Drawbridge deployment.

Tenant-scoped objects may include:

- Devices;
- User mappings;
- enrollment;
- Resources;
- administrators;
- Records visibility;
- shared-service relationships.

Cross-Tenant access requires explicit policy.

### Authority Grant

A current, explicitly scoped grant describing what an identity may perform within exact Site/Tenant/operation/target/scope/time/policy context.

Human-readable roles may bundle Authority Grants, but role names are not themselves the complete authorization decision.

### Revocation Operator

An identity holding narrowly scoped authority to remove trust from explicitly permitted targets. Revocation authority does not inherently grant enrollment, recovery, policy, PKI, or broader administrative authority.

### Trusted Network

An administrator-defined enterprise network state that has been positively
validated according to Drawbridge trust policy.

An SSID, subnet, gateway address, or other unauthenticated network
characteristic is not by itself equivalent to Trusted Network status.

### Gateway Placement Profile

A versioned Site/deployment configuration object that defines the authorized
Drawbridge ingress and Gateway-placement path for a specific Device population
or trust/deployment domain.

A Gateway Placement Profile defines at least:

- primary Drawbridge Service Address;
- authorized Front Distributor/ingress set;
- eligible Gateway pool;
- total Front Distributor failure behavior;
- any explicitly authorized secondary ingress;
- any explicitly authorized direct-Gateway fallback targets;
- deterministic target ordering/selection rules where fallback has multiple targets.

Domain-Managed and Shared-Service deployments may use separate Gateway Placement
Profiles, Service Addresses, Front Distributor sets, and Gateway pools. Failure
does not authorize crossing from one profile/trust domain into another.

Permitted total-ingress-failure behaviors include:

- FAIL_CLOSED;
- SECONDARY_INGRESS;
- DIRECT_GATEWAY_FALLBACK.

DIRECT_GATEWAY_FALLBACK changes only the transport destination. It does not
weaken Device/User/Tenant/Resource authorization or Gateway peer validation.
Only Gateways explicitly prepared, exposed, tested, and named by the profile
may receive direct fallback traffic.

## 5. Canonical relationship

```text
USER
  |
  | operates
  v
DEVICE / MDT
  |
  | runs
  v
DRAWBRIDGE AGENT
  |
  | establishes and maintains
  v
DEVICE SESSION
  |
  | over current
  v
TRANSPORT PATH
  |
  v
DRAWBRIDGE SERVICE ADDRESS
  |
  v
DRAWBRIDGE FRONT DISTRIBUTOR
  |
  | traffic placement
  v
DRAWBRIDGE GATEWAY
  |
  | independently authorized traffic
  v
ENTERPRISE FIREWALL / NETWORK
  |
  v
RESOURCE


                  DRAWBRIDGE CONTROLLER
                 configuration / policy
                  identity / Records
                         |
              +----------+----------+
              |                     |
              v                     v
       DRAWBRIDGE AGENT      DRAWBRIDGE GATEWAY
```

A User may add a User Session to an existing Device Session:

```text
Device authenticated
    ->
Device Session established
    ->
restricted Device/bootstrap access

User authenticated and authorized
    ->
User Session established
    ->
additional explicitly authorized Resources
```

## 6. Deprecated ambiguous terms

Do not use the following as standalone Drawbridge architecture terms:

```text
Drawbridge Client
Drawbridge Edge
client
edge
```

Historical documents or discussions may contain:

```text
Drawbridge Client -> Drawbridge Agent
Drawbridge Edge   -> Drawbridge Gateway
```

The word `client` may still appear where it is part of a precise external
technical term, for example:

```text
X.509 client authentication
TLS client authentication
```

In those cases it describes the external protocol role, not a Drawbridge actor.

## 7. Identifier guidance

Prefer identifiers that preserve the same distinctions:

```text
user_id
device_id
agent_version
front_distributor_id
gateway_id
device_session_id
user_session_id
transport_path_id
resource_id
site_id
tenant_id
gateway_placement_profile_id
recovery_store_id
authority_grant_id
```

Avoid ambiguous architecture identifiers such as:

```text
client_id
edge_id
```

unless an external protocol or API uses that exact term and the boundary is
explicitly documented.

## 8. Final terminology rule

Do not infer one Drawbridge actor from another.

These remain distinct:

```text
User                    != Device
Device                  != Drawbridge Agent
Drawbridge Agent        != Drawbridge Gateway
Device Session          != User Session
Transport Path          != Device identity
Drawbridge Service Address != peer authentication
Gateway Placement Profile != security authorization
Front Distributor placement != Gateway authorization
Gateway authorization   != enterprise firewall authorization
Domain identity          != domain administrative authority
Certificate validity     != current Drawbridge authorization
Historical validity      != current authority
Network connectivity     != application authorization
```

When a design statement, Record, API field, error, test, or policy decision
depends on one of these distinctions, name the exact actor or object.
