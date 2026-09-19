# Drawbridge Roadmap

This roadmap is architectural, not a delivery commitment.

## Phase 0 - Contracts and design

Freeze:

- threat/compromise model;
- orthogonal authoritative state domains;
- Domain Agent and Shared-Service Agent contracts;
- Agent/Gateway/Controller/Connector privilege boundaries;
- device/user authorization contract;
- DNP-style Authority Grant model;
- immutable-from-birth Records contract;
- Drawbridge Recovery Store contract;
- policy object model;
- configuration/version model;
- test/promotion model;
- PKI profiles and PKI-compromise behavior;
- Site/Tenant model;
- stable Drawbridge Service Address and Front Distributor authority boundary;
- HA/session-continuity contract;
- transport requirements;
- licensing invariants.

Deliverable: design baseline suitable for implementation review.

Current Phase 0 status and dependency-ordered remaining work are tracked in [PHASE-0-REVIEW.md](PHASE-0-REVIEW.md).

## Phase 1 - Basic lab transport

Build:

- separate Windows Domain Agent and Shared-Service Agent compositions;
- narrow privileged Network Helper proof-of-concept where required;
- Drawbridge Front Distributor proof-of-concept behind a stable Service Address;
- Drawbridge Gateway proof-of-concept;
- basic Gateway health/placement contract;
- virtual interface;
- stable virtual address;
- secure authenticated transport;
- basic IP forwarding;
- simple explicit resource policy.

Success:
- Windows endpoint reaches a protected subnet through Drawbridge.

## Phase 2 - Device identity

Build:

- machine certificate authentication;
- domain-managed enrollment path;
- shared-services PKI path;
- device authorization;
- certificate lifecycle basics;
- device-management plane.

Success:
- endpoint can establish restricted device connectivity before user authorization.

## Phase 3 - User authorization

Build:

- AD group authorization;
- optional RADIUS/NPS;
- initial federated/shared-services user identity;
- separate user access plane.

Success:
- authorized user receives resource policy without changing the device identity.

## Phase 4 - Mobility

Build:

- path migration;
- NAT rebinding;
- suspend/resume;
- Wi-Fi/Ethernet/LTE transition testing;
- dead-zone behavior;
- transport Records.

Success:
- logical session and virtual identity survive realistic network changes.

## Phase 5 - Trusted networks

Build:

- secure trusted-network classification;
- overlay bypass/suspend;
- clean transition into/out of enterprise Wi-Fi/wired networks.

Success:
- operational application session survives expected transition scenarios.

## Phase 6 - Records

Build:

- immutable-from-birth canonical object format and durable commit acknowledgement;
- rebuildable derived indexing/search;
- endpoint sensor baseline informed by Pathfinder UDM-Pro sensor;
- connection attempts;
- DNS context;
- process attribution;
- policy decisions;
- Gateway observations;
- network transitions;
- location observation with source/accuracy/freshness where platform capability permits;
- correlation.

Success:
- reconstruct a complete connection/session timeline.

## Phase 7 - Test/staging and versioned policy

Build:

- candidate configuration;
- mandatory comments;
- diff;
- test deployment;
- validation records;
- exact-artifact promotion;
- Controller push-and-verify execution receipts;
- monotonic rollback/revert as a new version;
- counterfactual Records replay.

Success:
- operational stakeholders can validate a candidate and promote exactly what they tested.

## Phase 8 - HA and production operations

Build:

- redundant Front Distributor tier;
- stable Service Address failover;
- Front Distributor drain/failure handling;
- Gateway clustering;
- session resume across Gateway failure;
- Controller independence;
- upgrade preflight;
- Drawbridge Recovery Store deployment and restore testing;
- domain-compromise recovery exercise;
- DR;
- health/self-review model;
- synthetic functional tests.

Success:
- planned maintenance and single-node failure do not create mass outage.

## Phase 9 - Pilot hardening

Focus on:

- Windows/MDT behavior;
- public-safety workflow;
- performance;
- transport reliability;
- ugly networks;
- sleep/resume;
- carrier transitions;
- MTU;
- DNS;
- upgrade lifecycle;
- support diagnostics.

Pilot should begin small and expand only after operational acceptance.

## Deferred / explicitly not v1

Do not allow scope creep into:

- DLP;
- CASB;
- full SWG;
- EDR;
- malware inspection;
- browser isolation;
- generic SIEM replacement;
- autonomous policy changes.

These are outside the core Drawbridge problem unless later justified.
