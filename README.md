# Drawbridge

**Drawbridge by Iron Signal Systems**

Drawbridge is a mobility-first remote-access platform intended to preserve secure, authorized network access across changing or unreliable networks while remaining understandable to network and systems administrators.

Drawbridge is a separate Iron Signal Systems product. It may integrate closely with Stronghold, but it must remain independently deployable behind conventional enterprise firewalls and routing environments.

## Product statement

> Drawbridge provides a controlled, persistent path between authorized devices, users, and enterprise services across changing or untrusted networks.

Drawbridge is not intended to be "another VPN." Its core design goals are:

- persistent device and user connectivity across network changes;
- clear separation of device identity, user authorization, network state, and application authorization;
- Active Directory and enterprise PKI as first-class integration points for domain-managed Windows environments;
- a separate shared-services trust model for non-domain devices and agencies;
- deterministic access policy that a network administrator can explain;
- first-class test/staging and promotion workflows;
- Git-like, attributable production change history;
- forensic-quality Records of connection attempts, DNS context, network changes, policy decisions, location observations, and gateway actions;
- licensing that can never become a packet-forwarding dependency;
- site-based licensing rather than per-user or per-device seat counting.

## Design principles

1. **Connectivity is not trust.**
2. **The device establishes the transport; the user receives authorization.**
3. **Commercial licensing never terminates authorized production connectivity.**
4. **Production changes are versioned, commented, tested, attributable, and reversible.**
5. **The exact configuration tested is the configuration promoted.**
6. **Test/staging is a first-class feature, not an optional lab SKU.**
7. **The client stays out of the user's way when healthy.**
8. **The infrastructure does not stay out of the administrator's way when something is wrong.**
9. **A connected tunnel is not proof that required services are functional.**
10. **Records are a first-class subsystem, not a reporting add-on.**
11. **Historical observations are preserved; later conclusions do not rewrite prior observations.**
12. **Shared-service connectivity does not imply domain trust or endpoint administrative authority.**
13. **Drawbridge should make secure change safer than doing nothing.**
14. **The internal policy model should be simple and deterministic even when external integrations are broad.**

## Initial target

The first serious target is a Windows-centric enterprise/public-safety deployment:

- Windows client;
- Drawbridge Controller;
- redundant Drawbridge Edge nodes;
- Active Directory integration;
- AD CS / domain PKI integration;
- optional NPS/RADIUS integration;
- shared-services mode using Drawbridge-managed device PKI;
- trusted-network detection;
- persistent mobility session;
- device-management plane;
- user authorization plane;
- deterministic access policies;
- Records;
- test/staging;
- versioned promotion and rollback.

## Documents

- [AGENTS.md](AGENTS.md)
- [LICENSE](LICENSE)
- [ARCHITECTURE.md](ARCHITECTURE.md)
- [IDENTITY-AND-TRUST.md](IDENTITY-AND-TRUST.md)
- [POLICY-AND-STATE.md](POLICY-AND-STATE.md)
- [ROUTING-AND-DEPLOYMENT.md](ROUTING-AND-DEPLOYMENT.md)
- [CLIENT-SENSOR-AND-DNS.md](CLIENT-SENSOR-AND-DNS.md)
- [RECORDS.md](RECORDS.md)
- [TESTING-AND-CHANGE-CONTROL.md](TESTING-AND-CHANGE-CONTROL.md)
- [LICENSING.md](LICENSING.md)
- [OPERATIONS.md](OPERATIONS.md)
- [ROADMAP.md](ROADMAP.md)
- [OPEN-QUESTIONS.md](OPEN-QUESTIONS.md)
- [REVIEW-CHECKLIST.md](REVIEW-CHECKLIST.md)

## Status

This repository baseline is a design package only. It intentionally contains no production code yet.

No implementation should be treated as approved until the architecture is reviewed and accepted.
