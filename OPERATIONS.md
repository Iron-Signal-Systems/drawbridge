# Operations

## 1. Client philosophy

When healthy, Drawbridge should remain mostly invisible to the end user.

The product should preserve the desirable part of mature mobility clients: officers and field users should not troubleshoot networking during normal work.

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
- current Edge;
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

- expiring CA;
- expiring Edge certificate;
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

Controller/Edge should support representative functional checks such as:

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
