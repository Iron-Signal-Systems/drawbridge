# Drawbridge Roadmap

This roadmap is architectural, not a delivery commitment.

## Phase 0 - Contracts and design

Freeze:

- trust model;
- state machine;
- device/user authorization contract;
- Records schema;
- policy object model;
- configuration/version model;
- test/promotion model;
- PKI profiles;
- site/tenant model;
- transport requirements;
- licensing invariants.

Deliverable: design baseline suitable for implementation review.

## Phase 1 - Basic lab transport

Build:

- Windows Drawbridge Agent proof-of-concept;
- Drawbridge Gateway proof-of-concept;
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

- endpoint sensor baseline informed by Pathfinder UDM-Pro sensor;
- connection attempts;
- DNS context;
- process attribution;
- policy decisions;
- Gateway observations;
- network transitions;
- optional location observation;
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
- rollback/revert;
- counterfactual Records replay.

Success:
- operational stakeholders can validate a candidate and promote exactly what they tested.

## Phase 8 - HA and production operations

Build:

- Gateway clustering;
- session resume across Gateway failure;
- Controller independence;
- upgrade preflight;
- DR;
- health model;
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
