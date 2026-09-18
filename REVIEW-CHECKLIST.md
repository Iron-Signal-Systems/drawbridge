# Drawbridge v0 Review Checklist

## Product boundary

- [ ] Drawbridge remains separate from Stronghold.
- [ ] Stronghold integration is optional.
- [ ] Drawbridge is mobility/remote access, not an enterprise firewall.
- [ ] Windows/public-safety is the initial gold path.

## Threat model and privilege

- [ ] Domain Agent and Shared-Service Agent are distinct implementations/contracts.
- [ ] Agent privilege is read-oriented and any privileged Network Helper is narrow.
- [ ] Agents do not trust/manage peer Agents.
- [ ] Gateway/Controller/Connector service identities are host scoped.
- [ ] Separate functions use separate gMSAs where permissions differ.
- [ ] Gateway identities have no general domain authority.
- [ ] Controller channels are initiated by managed components and carry declarative artifacts, not generic remote commands.
- [ ] Controller management/control networking is isolated from ordinary traffic.
- [ ] Service Connectors are Resource scoped and independently firewalled.
- [ ] Full domain compromise is treated as domain trust collapse.

## Identity

- [ ] Domain-managed devices use configured Mobility OUs.
- [ ] Domain-managed users use configured AD groups.
- [ ] Domain PKI machine certificate establishes device/bootstrap connectivity.
- [ ] NPS/RADIUS is optional.
- [ ] Shared-service devices do not authenticate to the hosting agency AD.
- [ ] Shared-service devices use explicit Drawbridge/shared-services PKI.
- [ ] Shared-service users may use federation/RADIUS/local strong identity.
- [ ] Application authentication remains separate where appropriate.
- [ ] Certificate validity alone does not mean current Drawbridge authorization.
- [ ] PKI compromise blocks all new trust establishment through the affected issuer.
- [ ] One authorized in-scope Revocation Operator may immediately revoke a lost/stolen Device.
- [ ] Recovery requires hands-on IT verification and full certificate re-issue.
- [ ] Administrative authorization uses scoped Authority Grants rather than role-name checks alone.

## Access and routing

- [ ] Device connectivity and user access are separate.
- [ ] Shared-service inbound management defaults to deny.
- [ ] Remote administration requires explicit policy and endpoint credentials.
- [ ] Trusted enterprise networks may bypass/suspend the overlay.
- [ ] Split/FQDN routing is base functionality.
- [ ] Routed DMZ handoff can leave enterprise routing with the existing firewall.
- [ ] Full tunnel remains available.
- [ ] NAT is compatibility behavior, not the preferred default.
- [ ] Service connector/proxy mode is available for complex deployments.

## Agent sensor and DNS

- [ ] Every endpoint includes a first-class sensor.
- [ ] Connection attempts, DNS context, route decisions, and policy decisions are recorded.
- [ ] DNS behavior is explicit Drawbridge policy.
- [ ] Local DNS may be used where policy allows.
- [ ] Protected namespaces may use private/Drawbridge DNS while public DNS remains local.
- [ ] All-DNS-through-Drawbridge mode is available.
- [ ] Hybrid resolver policy is available.
- [ ] Protected private namespaces do not leak to untrusted local DNS by default.
- [ ] FQDN authorization follows observed DNS answers and TTL/refresh state.
- [ ] Unauthorized DoH/DoT/hard-coded DNS can be controlled.

## Mobility

- [ ] Stable logical identity survives physical network changes.
- [ ] Roaming/dead-zone session persistence is core.
- [ ] Connected transport is not considered proof of functional service.
- [ ] HA/session recovery is required before serious pilot.

## Records

- [ ] Records is first-class.
- [ ] Pathfinder sensor philosophy is the baseline.
- [ ] Denied connection attempts are recorded.
- [ ] DNS resolution-at-time is preserved.
- [ ] Agent and Gateway observations are separate.
- [ ] Policy version and configuration commit are linked to decisions.
- [ ] Location is supported but strongly controlled.
- [ ] Canonical Records objects are immutable from birth.
- [ ] Canonical objects are complete write-once objects, not append targets.
- [ ] Mutable indexes/views are derived and rebuildable.
- [ ] Complete revocation/recovery and authority-exercise lifecycles are recorded.

## Change control and testing

- [ ] Every production-affecting change is versioned.
- [ ] Human reason/comment is mandatory.
- [ ] Test/staging is first-class and included.
- [ ] Exact tested artifact is promoted.
- [ ] PD Ops / 911 / EMO validation can be recorded.
- [ ] Emergency changes are auditable and reviewed.
- [ ] Rollback creates a new higher production version rather than activating an old artifact.
- [ ] Managed components verify artifact hash/version and report effective-state verification.
- [ ] Production Records can be replayed against candidate policy.

## Recovery Store

- [ ] DRS is independent of production AD trust.
- [ ] Complete snapshots are created after production changes and at least every 12 hours.
- [ ] DRS canonical objects are immutable from birth.
- [ ] Network-facing VNET jails cannot modify canonical storage.
- [ ] Host-side Sealer has no network listener.
- [ ] Recovery access is read-only and independently authenticated.
- [ ] Acceptance includes a test rebuild from a verified DRS object.

## Licensing

- [ ] Site licensing is the preferred model.
- [ ] Endpoint/user counts do not control forwarding.
- [ ] Test/DR are included.
- [ ] Commercial license failure never terminates authorized traffic.
- [ ] Reboot/patch/failover cannot turn commercial license state into an outage.
- [ ] Security revocation remains separate and enforceable.

## Operations

- [ ] Healthy Agent remains mostly out of the user's way.
- [ ] IT gets detailed state and diagnostics.
- [ ] Latent fatal states are detected before patch/reboot/upgrade where possible.
- [ ] Upgrade preflight is mandatory.
- [ ] Device refresh can overlap safely.
- [ ] IPv6 is included from design start.
- [ ] DNS and overlapping RFC1918 space are explicit design problems.
- [ ] Break-glass recovery is controlled and audited.

## Scope restraint

- [ ] DLP is not v1.
- [ ] CASB is not v1.
- [ ] SWG is not v1.
- [ ] EDR is not v1.
- [ ] Drawbridge does not autonomously change production policy.
