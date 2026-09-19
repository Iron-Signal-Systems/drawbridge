# Drawbridge Enforcement Boundaries

## 1. Purpose

This document defines the Phase 0 enforcement contract between the Drawbridge Agent, Drawbridge Front Distributor, Drawbridge Gateway, optional Service Connector, and the customer's enterprise firewall/network.

Its purpose is to prevent implementation from turning routing observations, endpoint claims, traffic placement, or one component's policy decision into authority owned by another component.

The central rule is:

> A component may enforce only the authority assigned to it. A broader decision at one layer never overrides a restriction at another layer.

For tunneled enterprise traffic, effective forwarding is the restrictive intersection of:

```text
Agent local permission/path selection
AND
Gateway independent authorization
AND
enterprise firewall/network acceptance
```

The destination application may apply additional authentication/authorization after network connectivity is established.

## 2. Action meanings

Drawbridge uses precise action semantics.

### Agent actions

The Agent may produce endpoint-side actions such as:

- DIRECT — policy permits the flow to use the Device's normal local network path;
- TUNNEL — policy selects the Drawbridge transport for the flow;
- DENY — the Agent blocks the flow locally.

TUNNEL means only:

> this flow is eligible to be presented to a Drawbridge Gateway for independent authorization.

TUNNEL does not mean ALLOW at the Gateway.

DIRECT means only that Drawbridge permits the local/direct path. It does not bypass endpoint OS/firewall controls or make an external destination trustworthy.

### Front Distributor action

The Front Distributor performs PLACEMENT:

> send this Drawbridge transport to this eligible Gateway.

PLACEMENT is not authorization.

### Gateway actions

The Gateway independently produces an enforcement result such as:

- ALLOW_FORWARD;
- DENY;
- LIMITED/QUARANTINED behavior where explicitly defined by policy.

The Gateway never converts an Agent TUNNEL decision into ALLOW merely because the Agent requested it.

### Enterprise firewall/network action

The enterprise firewall/network independently routes, filters, segments, NATs, or denies according to the customer's network policy.

Drawbridge ALLOW_FORWARD does not mean the enterprise firewall must allow the traffic.

## 3. Drawbridge Agent boundary

The Agent is the first Drawbridge enforcement point on the Device.

It may:

- observe local connection attempts;
- observe DNS questions/answers and resolver selection;
- observe process/User context to the degree supported by the platform;
- classify the current local network;
- select DIRECT versus TUNNEL;
- deny locally disallowed traffic;
- enforce protected-namespace DNS behavior;
- enforce local inbound Drawbridge policy;
- attach correlation/context metadata to tunneled traffic;
- apply the Device's current signed/versioned policy and Gateway Placement Profile.

The Agent may use local context to narrow access or choose a path.

The Agent must not:

- grant final enterprise Resource authorization;
- manufacture a current User Session;
- choose an arbitrary Tenant;
- choose an arbitrary Gateway pool outside its Gateway Placement Profile;
- treat certificate possession alone as current Device authorization;
- treat process, location, SSID, subnet, DHCP, DNS, or gateway address as sufficient proof for broader server-side authorization;
- bypass a Gateway deny;
- silently route a protected enterprise Resource DIRECT because the tunnel path is unavailable.

A compromised Agent may falsify Device-originated observations and may control traffic originating from that compromised Device. That does not give the Agent authority over peer Devices, Gateway policy, Controller policy, the enterprise firewall, or other Tenants.

## 4. Front Distributor boundary

The Front Distributor is a traffic-placement component.

It may use only information required for placement, such as:

- ingress/Service Address context;
- Gateway Placement Profile context that is valid for the ingress/session;
- transport metadata required for routing;
- Gateway health/readiness;
- capacity;
- maintenance/drain state;
- site affinity;
- a validated transport-routing token or Connection ID if the selected transport design supports one.

A client-supplied profile name, Tenant name, Gateway name, or routing hint is not authority by itself.

The Front Distributor must not:

- grant Device, User, Tenant, or Resource authorization;
- compile or alter policy;
- create User Session assertions;
- select a different trust/deployment profile merely because it is reachable;
- turn DIRECT_GATEWAY_FALLBACK into arbitrary Gateway discovery;
- administer AD/PKI/endpoints;
- become the sole owner of irreplaceable Device Session state.

If the Front Distributor terminates an outer transport layer, that termination is not sufficient Device/session authentication. The selected Gateway must still independently validate the Drawbridge Device/session security context.

## 5. Drawbridge Gateway boundary

The Gateway is the authoritative Drawbridge enforcement point for traffic entering the enterprise through Drawbridge.

Before forwarding a new flow, the Gateway independently validates the security facts required by policy, including as applicable:

- authenticated Device/session binding;
- current Device authorization and revocation state;
- current User Session authorization where the Resource requires a User;
- Tenant binding;
- Resource destination;
- protocol/port/direction;
- current policy/configuration generation;
- Gateway Placement Profile compatibility where relevant;
- current PolicyPosture;
- authoritative FQDN/DNS binding where FQDN policy is used;
- inbound/return-flow state;
- other security inputs defined by the deterministic policy pipeline.

The Gateway does not trust an Agent's TUNNEL decision as authorization.

The Gateway may consume Agent observations for correlation and for policy inputs whose assurance contract explicitly permits it. An Agent-provided observation must not broaden access beyond independently authoritative Gateway state unless a later design contract defines how that observation is positively verified.

Examples:

- process attribution is initially observational and may narrow local Agent behavior, but does not independently grant a broader Gateway Resource authorization;
- Device location is an observation, not proof of authorization;
- Agent-reported network state may not broaden Gateway authorization merely because the Agent says a network is trusted.

## 6. Resource policy at Agent and Gateway

For tunneled enterprise Resources, Agent and Gateway enforcement are defense in depth, not mutual trust.

The Agent should block or avoid sending traffic that its current policy does not permit.

The Gateway must independently reject traffic its current policy does not permit even if the Agent sends it.

Therefore:

```text
Agent DENY
    -> final local DENY

Agent TUNNEL
Gateway DENY
    -> final DENY

Agent TUNNEL
Gateway ALLOW_FORWARD
Firewall DENY
    -> no successful enterprise connectivity

Agent TUNNEL
Gateway ALLOW_FORWARD
Firewall ALLOW
Application DENY
    -> network path exists; application access still denied
```

A compromised or stale Agent cannot turn a Gateway DENY into ALLOW.

A stale or misconfigured Gateway cannot force an Agent to originate traffic the Agent locally denies.

## 7. Policy-generation skew and restrictive intersection

Agent and Gateway policy/configuration may be temporarily at different generations during controlled distribution, recovery, or failure.

Version skew must never create a transient broader grant.

Rules:

- a new access grant becomes usable only when every required enforcement point has a compatible policy that permits it;
- a deny/removal at either required enforcement point is sufficient to stop new matching traffic at that point;
- current revocation/security-urgent state overrides stale allow state when known;
- unknown/incompatible policy generation does not invent authorization;
- mismatched generations and resulting denies are recorded and visible operationally.

This makes rollout behavior a restrictive intersection rather than a race to the most permissive node.

## 8. DNS/FQDN authorization boundary

The Agent may use its observed DNS state to select TUNNEL versus DIRECT and to create Records.

For Gateway Resource authorization, an arbitrary Agent statement such as:

```text
cad.county.gov resolved to 10.40.20.15
```

is not by itself sufficient to broaden server-side access to 10.40.20.15.

The Gateway requires a current FQDN-to-address binding from an authority/observation path defined by the DNS architecture, such as a Gateway-observed protected DNS exchange, a Drawbridge-controlled resolver/binding service, or another explicitly defined verifiable mechanism.

The exact mechanism remains an implementation/design item, but the authority rule is locked:

> unverified Agent DNS claims may inform Records and local routing; they do not independently expand Gateway Resource authorization.

## 9. Enterprise firewall/network boundary

The existing enterprise firewall remains an independent security and routing boundary.

It remains responsible for customer-defined functions such as:

- zone/inter-segment policy;
- internal routing;
- NAT where desired;
- Internet egress policy;
- conventional network security controls;
- Connector-to-Resource restrictions where Service Connectors are used.

Drawbridge should preserve the virtual Device source identity/address through routed handoff where practical so the enterprise firewall can continue applying understandable network policy.

Drawbridge must not silently widen firewall rules, disable segmentation, or create broad NAT/exposure merely to make a Drawbridge flow succeed.

Unless an explicit integration exists, Drawbridge must not claim that the enterprise firewall knows the Drawbridge User identity merely because it can see a virtual Device address.

## 10. Service Connector boundary

Where service-proxy/Connector handoff is used, the Service Connector is a Resource-scoped bridge after Gateway authorization.

It may:

- accept only authenticated Drawbridge service requests from the authorized Gateway/service channel;
- connect only to explicitly assigned backend Resources/supporting services;
- preserve Tenant/Resource correlation for Records;
- deny requests outside its configured backend/resource contract.

It must not:

- turn a Gateway allow into general internal-network reachability;
- choose arbitrary backend hosts because the Agent requested them;
- become a jump host or administrative relay;
- create broader Tenant/Resource authorization;
- bypass the enterprise firewall.

For Connector mode, effective service reachability is still restrictive:

```text
Agent local permission/path selection
AND
Gateway independent authorization
AND
Service Connector configured Resource scope
AND
enterprise firewall/network acceptance
```

A Connector may further deny a flow; it never broadens what the Gateway authorized.

## 11. Inbound and return traffic

Return traffic and unsolicited inbound traffic are different.

### Return traffic

Return traffic is permitted through Drawbridge only when it belongs to an authorized flow/session according to the Gateway's connection/session state and policy.

The Agent/Device endpoint remains subject to its local OS/firewall behavior.

### Unsolicited inbound / remote management

Unsolicited inbound connectivity is denied by default unless an explicit Remote Management Policy authorizes it.

An authorized inbound management path requires the restrictive intersection of:

- permitted enterprise source/Resource;
- permitted protocol/port/direction;
- Gateway inbound authorization;
- Agent local inbound authorization;
- endpoint OS/application authentication;
- enterprise firewall permission.

Domain-Managed Devices may permit explicit inbound management according to policy.

Shared-Service Devices default to no inbound management. Enabling it requires explicit policy and does not grant the hosting agency local administrator rights.

## 12. Domain-Managed versus Shared-Service enforcement

The same core enforcement equation applies to both modes, but their bootstrap and trust authorities differ.

### Domain-Managed

Before User authorization, the Device plane may permit only explicitly defined bootstrap/management Resources such as:

- enterprise DNS;
- required AD/Kerberos/LDAP/LDAPS services;
- SYSVOL/NETLOGON where required;
- NTP;
- PKI/CRL/OCSP;
- Drawbridge control services;
- approved endpoint-management services.

Business Resources requiring a User remain denied until a valid User Session exists.

### Shared-Service

A Shared-Service Device is not granted hosting-domain trust merely because it can reach a hosted service.

Its pre-User/device plane is limited to explicitly required Drawbridge, enrollment/PKI, home/federated authentication, and other configured supporting Resources.

Gateway and optional Service Connector enforcement must preserve Tenant/Agency isolation. Cross-Tenant access requires explicit policy.

## 13. Observation disagreement

Agent and Gateway observations are intentionally independent.

Examples:

```text
Agent said TUNNEL
Gateway saw no flow
    -> transport/placement problem or false Agent claim

Agent said SENT
Gateway received
Gateway DENIED
    -> Gateway policy/security decision

Agent said destination A
Gateway observed destination B
    -> deny where policy requires exact match and record a security-relevant mismatch

Agent reported process X
Gateway cannot independently prove process X
    -> preserve as Agent observation, not Gateway fact
```

When observations disagree:

- do not rewrite one side to match the other;
- do not silently choose the more permissive interpretation;
- preserve both facts with producer identity and time;
- deny where the disagreement makes authorization ambiguous;
- create a security/health Record when the mismatch is meaningful.

## 14. Records requirements

Enforcement Records should make the chain reconstructable:

- Agent observation;
- Agent path decision and policy generation;
- Gateway Placement Profile/version;
- Front Distributor placement;
- Gateway receipt;
- Gateway authorization inputs;
- Gateway decision and policy generation;
- enterprise handoff;
- firewall/network result only when Drawbridge actually observes or receives that result;
- application result only when Drawbridge actually observes it.

Drawbridge must preserve these truth separations:

```text
Agent TUNNEL != Gateway ALLOW_FORWARD
Front Distributor PLACEMENT != Gateway authorization
Gateway ALLOW_FORWARD != enterprise firewall allow
enterprise firewall allow != application authorization
Agent said sent != Gateway received
```

## 15. Locked versus open

Locked by P0-03:

- Agent owns local observation, DIRECT/TUNNEL/DENY selection, and local enforcement;
- TUNNEL is a request for independent Gateway authorization, not an allow;
- Front Distributor owns placement only;
- Gateway independently authenticates/authorizes tunneled enterprise traffic;
- Agent and Gateway permissions combine restrictively;
- a broader decision at one enforcement point never overrides a deny at another;
- version skew cannot create a transient broader grant;
- unverified Agent process/location/network/DNS observations do not independently broaden Gateway authorization;
- enterprise firewall/network remains an independent deny/routing boundary;
- Service Connector mode is Resource scoped and can further restrict but never broaden Gateway authorization;
- unsolicited inbound is deny by default and requires independent Gateway + Agent + endpoint + firewall authorization;
- Shared-Service and Domain-Managed trust/bootstrap differences do not change the core enforcement equation;
- observation disagreements remain separate Records and ambiguous security state does not fail open.

Still owned by later Phase 0 items:

- exact User Session assertion/binding: P0-04;
- deterministic ordered policy evaluation: P0-05;
- PKI credential profiles: P0-06;
- canonical Record envelope: P0-07;
- transport mechanics: P0-09;
- Windows WFP/virtual-adapter enforcement implementation: P0-11;
- positive Trusted Network proof: P0-12;
- complete failure-mode thresholds: P0-15;
- exact authoritative DNS/FQDN binding mechanism: DNS/Agent design work.
