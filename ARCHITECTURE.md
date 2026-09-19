# Drawbridge Architecture

## 1. System role

Drawbridge provides secure, persistent remote connectivity without becoming the enterprise firewall.

Canonical component and actor names are defined in [TERMINOLOGY.md](TERMINOLOGY.md) and are normative for this architecture.

It must be able to operate with:

- Stronghold;
- FortiGate;
- Palo Alto;
- Cisco;
- other conventional firewalls;
- ordinary routed enterprise networks.

A Stronghold integration may provide richer identity and policy context, but Stronghold must not be required for Drawbridge to function.

## 2. Logical components

### Drawbridge Agent

Drawbridge has two explicit Agent contracts: Domain Agent and Shared-Service Agent. They may share narrow libraries but remain distinct top-level implementations.

The Agent's primary responsibilities are networking/transport and presentation of the Device credential while carrying required Windows/User authentication and MFA traffic.

The Domain Agent runs under a narrowly scoped gMSA. Elevated network operations, where required, use a narrow Network Helper rather than a general privileged Agent.

Agents do not trust, administer, or directly control peer Agents.

Common endpoint functions include:

- device authentication;
- virtual network identity;
- mobility transport;
- trusted-network detection;
- local traffic classification/enforcement;
- user-session authorization integration;
- local Records collection;
- network and location observations;
- offline policy cache;
- health reporting.

The Agent should remain unobtrusive to normal users. Operational detail is primarily for IT.

### Drawbridge Controller

Responsible for:

- configuration;
- policy compilation;
- directory synchronization;
- user/device entitlement;
- PKI enrollment workflow;
- site and tenant configuration;
- test/staging orchestration;
- version history;
- promotion/rollback;
- Records indexing/search;
- health and readiness status.

The Controller must not sit in the live packet forwarding path.

Managed components establish authenticated persistent control channels to the Controller. The Controller pushes declarative, versioned artifacts over those established channels; targets independently validate, apply, verify effective hash/state, and return execution receipts.

The Controller has no general-purpose remote shell into managed components and should operate on a dedicated management/control network.

### Drawbridge Gateway

Terminates remote Drawbridge transport and is responsible for:

- device/session authentication;
- virtual-address binding;
- packet forwarding;
- policy enforcement;
- session suspend/resume;
- path migration;
- Gateway-side Records;
- high-availability participation.

Each Gateway has host-specific gMSAs; separate functions use separate gMSAs where permissions differ. Gateway identities have no general domain or endpoint administrative authority.

### Records subsystem

Records is a peer of Identity, Mobility, and Policy.

It must preserve:

- what the endpoint observed;
- what the Gateway observed;
- what identity was presented;
- what policy was evaluated;
- what decision was made;
- what action was taken;
- what network/location state existed at the time;
- what configuration version produced the decision.

Canonical Records are complete write-once objects immutable from birth. Mutable indexes/views are derived and rebuildable.

### Drawbridge Recovery Store

DRS preserves complete, verifiable Gateway/Controller configuration outside production AD trust. A new immutable object is created after each production configuration change and at least every 12 hours.

Recovery-critical configuration for other components must be either deterministically regenerable from DRS-protected authoritative state or directly DRS-protected. DRS does not escrow reusable runtime secrets; recovery creates new identities/private credentials.

Preferred DRS design is hardened FreeBSD with ZFS, PF, VNET jails, and a host-local Sealer. See [RECOVERY-STORE.md](RECOVERY-STORE.md).

## 3. Core architecture

```text
                      DRAWBRIDGE CONTROLLER
                      configuration / policy
                       identity / records
                              |
                +-------------+-------------+
                |                           |
                v                           v
         DRAWBRIDGE AGENT            DRAWBRIDGE GATEWAY
                |                           |
                |<==== mobility transport =>|
                |                           |
          endpoint network            enterprise network
```

The Controller is authoritative for configuration and policy distribution but must not be required for every forwarded packet.

See [THREAT-MODEL.md](THREAT-MODEL.md) for normative compromise boundaries.

## 4. Device and user planes

Drawbridge separates two logical planes.

### Device / management plane

Exists after device authentication.

Purpose:

- allow the endpoint to remain manageable;
- support domain bootstrap where applicable;
- reach PKI and required infrastructure;
- maintain Drawbridge control connectivity.

For shared-service/non-domain devices, inbound management is denied by default.

### User access plane

Exists only after user authorization.

Purpose:

- allow the authorized user to reach explicitly assigned resources;
- apply user/resource/network policy;
- preserve user access across network changes where possible.

User disconnect or logout must not automatically destroy the device-management session.

## 5. Mobility model

The endpoint receives a stable Drawbridge virtual identity while the physical path may change.

```text
Virtual identity: stable

Underlying paths:
  Wi-Fi -> LTE/5G -> no network -> LTE/5G -> trusted Wi-Fi
```

Identity must not be bound to the endpoint's current external IP address.

The logical hierarchy is:

```text
device cryptographic identity
        |
        v
logical Drawbridge session
        |
        v
virtual address / virtual identity
        |
        v
current transport path
```

A path change must update the transport binding, not create a new device identity.

## 6. Trusted networks

Drawbridge must support administrator-defined trusted enterprise networks where the mobility overlay is unnecessary.

Trusted state must not depend solely on SSID.

Possible validation inputs include:

- successful enterprise 802.1X/EAP-TLS state;
- expected certificate trust;
- authenticated enterprise network responder;
- expected network/gateway characteristics;
- explicit site configuration.

On a trusted network:

- Drawbridge may suspend/bypass the remote overlay;
- normal enterprise network policy applies;
- Records continue;
- transition back to untrusted connectivity must be handled cleanly.

## 7. Multi-agency/shared-services model

Drawbridge must support agencies that share services without sharing:

- AD domains;
- endpoint administration;
- general network trust;
- user databases;
- local administrator rights.

A foreign/shared-service endpoint uses Drawbridge PKI for device identity and explicit resource policy.

The hosting agency does not gain endpoint administration merely because the device has network access.

## 8. Transport

The transport protocol is not yet frozen.

Candidates include:

- QUIC-based transport;
- CONNECT-IP over HTTP/3/QUIC;
- custom framing over a mature cryptographic transport.

Hard requirements:

- modern cryptography;
- replay protection;
- session identifiers independent of external IP;
- NAT rebinding support;
- path migration;
- suspend/resume after loss of connectivity;
- packet sequencing/reordering handling;
- MTU/PMTUD handling;
- IPv4 and IPv6;
- clean failure and diagnostic semantics.

Drawbridge must not invent novel cryptography.

## 9. HA

Production design assumes at least two Gateway nodes.

Requirements:

- Gateway failure must not invalidate endpoint identity;
- sessions must be resumable on another authorized Gateway;
- maintenance must not require mass reauthentication where avoidable;
- Controller loss must not immediately terminate established sessions;
- DR and test environments are part of the licensed site model.

## 10. Failure philosophy

A service is not "healthy" merely because:

- a process is running;
- a port is listening;
- the Agent reports connected;
- a tunnel exists.

Health must distinguish:

- control plane;
- device authentication;
- user authorization;
- policy engine;
- data plane;
- Records;
- PKI;
- directory integration;
- service reachability.

Drawbridge should support synthetic functional tests of representative traffic.

## 11. Routing/deployment flexibility

Routing must be a deployment choice.

Drawbridge supports two independent decisions:

- endpoint traffic selection: selective/split, full tunnel, trusted-network bypass;
- enterprise handoff: routed DMZ, optional NAT, or internal service proxy/connector.

The preferred simple deployment places Drawbridge Gateway in a DMZ and routes the Drawbridge virtual Device address pool through the customer's existing firewall. The existing firewall remains authoritative for enterprise routing and segmentation.

Complex proxy/connector deployments remain available where shared services, overlapping address space, or security boundaries make ordinary routed handoff unsuitable.

See [ROUTING-AND-DEPLOYMENT.md](ROUTING-AND-DEPLOYMENT.md).

## 12. Agent sensor and DNS control

Every Drawbridge Agent includes a first-class sensor informed by the Pathfinder UDM-Pro sensor model.

The sensor observes endpoint connection attempts, DNS activity, process/user context, route decisions, network transitions, and Drawbridge policy outcomes for Records and troubleshooting.

DNS is a Drawbridge policy function. The administrator may choose:

- local DNS;
- protected-namespace-only Drawbridge DNS;
- forced Drawbridge DNS;
- hybrid resolver policy.

Protected namespaces must not leak to untrusted local resolvers unless explicitly permitted.

See [AGENT-SENSOR-AND-DNS.md](AGENT-SENSOR-AND-DNS.md).
