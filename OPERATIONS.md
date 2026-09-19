# Operations

## 1. Agent and user-experience philosophy

When healthy, Drawbridge should remain mostly invisible to the end user.

The product should preserve the desirable part of mature mobility agents: officers and field users should not troubleshoot networking during normal work.

Suggested user states:

- Connected
- Device Only
- Trusted Network
- Limited
- Disconnected

IT receives the detailed state.

## 2. Administrator visibility

For an endpoint, IT should be able to see:

- device identity;
- certificate status;
- current user;
- authorization profile;
- physical interface;
- current external path;
- carrier/network where available;
- virtual address;
- current Gateway Placement Profile/version;
- current Front Distributor/ingress path;
- configured total-ingress-failure mode;
- current normal/fallback/degraded ingress mode;
- current Gateway;
- current session;
- trusted/untrusted state;
- policy version;
- route/resource decisions;
- transport latency/loss where measurable;
- last migration;
- Records spool health;
- location state according to policy.

## 3. No latent fatal state

Drawbridge should detect conditions likely to fail later during:

- reboot;
- patch cycle;
- certificate renewal;
- failover;
- upgrade;
- service restart.

Examples:

- stale/missing Drawbridge Recovery Store checkpoint;
- expiring CA;
- expiring Gateway certificate;
- failed CRL publication;
- virtual-address pool exhaustion;
- Records storage exhaustion;
- AD sync failure;
- unhealthy HA;
- incompatible versions;
- invalid policy reference;
- corrupted configuration;
- failed database checkpoint.

## 4. Functional health

"Connected" is not enough.

Health should distinguish:

- transport connected;
- device authenticated;
- user authorized;
- policy active;
- required resource reachability;
- control plane;
- data plane;
- Records;
- PKI;
- directory integrations.

A tunnel can be up while the system is operationally unusable. Drawbridge must expose that difference.

## 5. Synthetic tests

Controller/Gateway should support representative functional checks such as:

- policy lookup;
- route calculation;
- test-resource reachability;
- return traffic;
- DNS resolution;
- PKI validation;
- directory sync.

## 6. Remote support

Domain-managed device connection exists before/independent of user access where configured.

This allows IT to support:

- pre-logon issues;
- cached credential problems;
- certificate issues;
- GPO;
- software deployment;
- reboots;
- endpoint management.

For shared-service endpoints, remote administration remains denied unless explicitly granted.

## 7. Device lifecycle

Candidate lifecycle states:

- Active
- Staging
- Replacement
- Spare
- Repair
- Quarantined
- Retired
- Lost/Stolen

Security actions differ by state.

## 8. Device refresh

A replacement endpoint may coexist with the existing production endpoint while being validated.

Required workflow:

```text
old MDT ACTIVE
new MDT STAGING/REPLACEMENT
    ->
validate
    ->
complete replacement
    ->
new MDT ACTIVE
old MDT RETIRED/revoked as appropriate
```

If validation fails, the old MDT remains functional.

Site licensing makes seat transfer unnecessary.

## 9. DNS

DNS is a design-critical subsystem.

Requirements to resolve:

- domain bootstrap;
- split DNS;
- private service zones;
- tenant/agency DNS;
- DNS leak behavior;
- trusted-network behavior;
- FQDN policy consistency.

## 10. IPv6

IPv6 must be designed from the beginning.

Drawbridge must not create a secure IPv4 overlay while allowing uncontrolled IPv6 bypass.

## 11. Captive portals

Public/hotel Wi-Fi may require local portal access before Internet connectivity exists.

Drawbridge needs a tightly controlled captive-portal strategy that does not create a permanent broad bypass.

## 12. Overlapping private address space

Multi-agency deployments will encounter overlapping RFC1918 networks.

The architecture needs an explicit strategy before shared-services implementation.

## 13. Break-glass recovery

A bad policy must not permanently lock out the fleet.

Break-glass behavior should be:

- cryptographically authorized;
- time limited;
- heavily recorded;
- difficult to trigger accidentally;
- not a hidden backdoor account.

## 14. DNS operational visibility

For each endpoint, IT should be able to see:

- active DNS mode;
- current resolver(s);
- protected namespaces;
- last DNS decision;
- local versus Drawbridge DNS path;
- blocked alternate DNS attempts where applicable;
- DNS policy version.

DNS failure must be distinguishable from tunnel failure.


## 15. Lost/stolen Device response

Any authorized in-scope Revocation Operator may make Device revocation authoritative immediately. Reachable enforcement points terminate affected active Device Sessions, deny resume/new sessions, initiate certificate revocation, and record subsequent attempts. A truly partitioned enforcement point applies the authoritative revocation as soon as communication is restored.

Return to service requires hands-on IT verification, new key generation, full certificate re-issue, renewed enrollment, and validation.

## 16. Infrastructure/domain compromise recovery

Gateway, Controller, and Connector host-specific identities allow one node to be isolated without replacing healthy peer credentials.

Full domain compromise is treated as domain trust collapse. Recovery uses verified configuration from the independent Drawbridge Recovery Store, trusted-media rebuild, new identities/certificates, validation, and controlled return to production.

## 17. Controller self-review

The Controller compares its security posture to an explicit baseline and identifies unexpected services/listeners, broadened permissions, certificate/private-key ACL changes, widened firewall exposure, stale/revoked trust, configuration hash mismatch, and failed Records/DRS delivery. Self-review recommends remediation rather than silently broadening authority.


## 18. Dependency availability

Unavailable and compromised dependencies are different operational states.

Temporary Controller, Records, or DRS unavailability must not unnecessarily terminate independently valid established forwarding. Fresh authentication/authorization decisions do not silently bypass a required authority merely because it is unavailable.

DRS checkpoint age, Records spool pressure, control-channel backlog, and identity/PKI dependency availability are explicit health conditions.


## 19. Front Distributor and Gateway HA operations

Production ingress is the stable Drawbridge Service Address -> redundant Front Distributor tier -> eligible Gateway pool.

Front Distributor health must expose more than "port open." Operations need to distinguish at least:

- configuration current;
- ingress/listener functional;
- Gateway health view current;
- eligible for serving traffic;
- draining;
- degraded/not ready;
- failed.

Gateway placement eligibility must consider transport/listener health, control/config/policy currency, certificate/trust state, capacity, enterprise handoff, Records/spool pressure, and explicit maintenance state.

Operators must be able to drain a Front Distributor or Gateway before maintenance without changing endpoint configuration.

A single Front Distributor failure should be operationally boring: the stable service remains/re-becomes reachable through another ingress node and Agents resume/re-establish without redefining identity or authorization.

Loss of all Front Distributors follows the effective Gateway Placement Profile. Operators must be able to see whether the profile is FAIL_CLOSED, using SECONDARY_INGRESS, or operating in DIRECT_GATEWAY_FALLBACK. Direct fallback may use only explicitly prepared/named Gateways and deterministic configured selection; it must never cross Domain/Shared-Service profile boundaries or weaken normal Gateway authorization.

## 20. Gateway Placement Profile operations

Gateway Placement Profiles are normal production configuration and follow the same versioned/tested/promotion workflow as other production-affecting configuration.

Operational views should expose:

- profile name/ID and effective version;
- Device population/trust domain using the profile;
- primary Service Address;
- Front Distributor/ingress set;
- eligible Gateway pool;
- total Front Distributor failure behavior;
- secondary ingress where configured;
- direct-fallback Gateways and deterministic selection/order where configured;
- current normal/degraded/fallback mode;
- last profile-driven failover/fallback event.

A profile change that alters ingress or fallback authority is security- and availability-significant and must not be an anonymous runtime toggle.
