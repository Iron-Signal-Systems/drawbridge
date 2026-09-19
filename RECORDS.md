# Drawbridge Records

## 1. Records is a first-class subsystem

Drawbridge Records is not merely VPN logging or dashboard telemetry.

Its purpose is to reconstruct:

- what the endpoint attempted;
- what names were resolved at that moment;
- which process/user/device initiated activity;
- what network and location context existed;
- what policy evaluated the activity;
- what decision was made;
- what the Agent did;
- what the Gateway observed;
- what the Gateway did;
- how the session moved across networks;
- what configuration version was effective.

The Pathfinder UDM-Pro sensor philosophy is the baseline reference for Drawbridge's observation model.

Drawbridge should preserve the distinction between:

```text
what was observed
what was resolved at that time
what policy concluded
what later analysis concluded
```

## 2. Candidate record classes

- Flow / connection observation
- Connection attempt
- Connection established
- Connection failed
- Connection closed
- DNS question
- DNS answer / resolution-at-time
- Process/application observation
- Interface change
- Physical-network change
- Trusted-network classification
- Location observation
- Device authentication
- User authentication
- User authorization
- Agent path decision (DIRECT/TUNNEL/DENY)
- Gateway authorization decision
- Agent/Gateway policy-generation mismatch
- Agent/Gateway observation mismatch
- Policy decision
- Session established
- Session suspended
- Session resumed
- Session migrated
- Session terminated
- Gateway Placement Profile assignment/change
- Gateway Placement Profile fallback/degraded-mode event
- Front Distributor placement decision
- Front Distributor health/failover event
- Gateway eligibility/drain event
- Agent forwarding action
- Gateway receipt
- Gateway forwarding action
- Gateway deny/drop
- Certificate enrollment
- Certificate renewal
- Certificate revocation
- Directory sync event
- Configuration change
- Test deployment
- Validation result
- Production promotion
- Rollback/revert
- Software release/update verification
- Software installation/rollback result
- Records query/export/retention action
- Licensing/entitlement administrative event
- Health/preflight event

## 3. Failed and denied traffic matters

Drawbridge must record meaningful attempted communication even when policy prevents it from traversing the overlay.

Example:

```text
device: MDT-014
user: DOMAIN\jwood
process: powershell.exe
destination: 10.50.10.17:445/tcp
decision: DENY
reason: destination outside authorized resources
```

## 4. DNS context

Drawbridge should preserve the name-to-address relationship observed at the time.

Example:

```text
DNS question:
  cad.example.gov A

Observed answer:
  cad.example.gov -> 10.40.20.15

Connection:
  10.250.14.37:49182 -> 10.40.20.15:443

name_at_time:
  cad.example.gov
```

Historical records must not be rewritten because DNS resolves differently later.

## 5. Location

Location is a first-class recorded Drawbridge observation where the Device/platform can provide it.

Possible sources:

- GNSS;
- Windows Location Service;
- Wi-Fi positioning;
- cellular positioning;
- trusted-network/site classification;
- administrator-defined site identity.

Each location Record should preserve:

- source;
- timestamp;
- accuracy where available;
- age/freshness where available;
- the observation at the precision actually supplied by the source.

Physical location and network-derived location are separate facts and must not be silently conflated. Location is an observation with known source/quality, not infallible proof of Device position.

If a source cannot provide location, use an explicit state such as NOT_AVAILABLE, NOT_SUPPORTED, PERMISSION_DENIED, SENSOR_FAILED, or NOT_OBSERVED rather than an ambiguous semantic null where the distinction matters.

## 6. Location access and retention

Location collection is part of the normal Records model where platform capability permits it.

Access to location history is separately authorized, Tenant/Site scoped in backend policy, and auditable. Bulk export is a distinct sensitive operation where implemented.

Retention may differ from general connection metadata and is governed explicitly rather than inferred from canonical-object immutability.

## 7. Agent and Gateway independence

The Agent and Gateway should report independently.

Example correlation:

```text
AGENT OBSERVED:
  connection attempt

AGENT DECIDED:
  tunnel

GATEWAY OBSERVED:
  packet received

GATEWAY DECIDED:
  allow

GATEWAY DID:
  forwarded
```

This supports transport troubleshooting and forensic reconstruction.

## 8. Correlation identifiers

Candidate identifiers:

- `device_id`
- `session_id`
- `user_session_id`
- `connection_id`
- `policy_version`
- `config_commit_id`
- `gateway_id`
- `gateway_placement_profile_id`
- `tenant_id`
- `test_run_id`

## 9. Canonical objects are immutable from birth

A canonical object is created complete, validated, written once, verified, and never modified. There is no append, patch, update-in-place, or rewrite operation on a canonical object.

Corrections and later facts create new canonical objects linked to earlier objects where appropriate.

## 10. Integrity

The producer retains its local object until the receiver durably commits and acknowledges the exact object/hash. Same object ID plus same hash is idempotent retransmission; same ID plus different content is a security event.

Mutable search indexes, current-state views, caches, and dashboards are derived and rebuildable.

Integrity target:

- canonical record encoding;
- sequence/checkpointing;
- record/batch hashing;
- signed batches;
- deletion/insertion detection;
- verifiable archive.

This should be designed without requiring every customer to operate a courtroom-oriented system.

## 11. Retention

Retention should be configurable.

Illustrative tiers:

- hot/searchable;
- warm;
- archive.

Records should be exportable to external platforms without requiring customers to use a separate Drawbridge analytics SKU.

## 12. Operational use beyond forensics

Records should support safe policy reduction.

Example:

```text
Current rule:
  Agency-B -> 10.40.0.0/16

Observed for 180 days:
  10.40.20.17:443
  10.40.20.18:443

Potential reduction:
  /16
  ->
  two /32 HTTPS destinations
```

Drawbridge may recommend changes, but must not autonomously modify production access policy.

## 13. Counterfactual policy evaluation

Production Records should be replayable against a candidate policy.

Drawbridge should be able to answer:

> If this candidate policy had been active for the last N days, what would have changed?

Results should include:

- newly denied connections;
- affected devices;
- affected users;
- affected resources;
- connection counts;
- confidence/coverage limits.

## 14. Agent sensor baseline

The Drawbridge Agent sensor is a mandatory observation source.

At minimum it should correlate:

- connection attempt;
- process/user;
- DNS question;
- DNS answer;
- resolver selected;
- DNS policy decision;
- FQDN binding;
- route decision;
- tunnel/direct decision;
- current network;
- session state;
- policy version.

The Agent sensor should follow the same observation discipline as Pathfinder: preserve what was observed independently from later assessment.


## 15. Security lifecycle completeness

Lost/stolen and compromise incidents preserve reporting, revocation, session termination, certificate action, denied reconnects, hands-on recovery verification, replacement credential issuance, re-enrollment, validation, and return to service as separate immutable-from-birth canonical objects.


## 16. HA and distribution Records

Front Distributor/Gateway placement and failover must be reconstructable rather than disappearing as infrastructure detail.

Relevant Records should preserve, where applicable:

- Gateway Placement Profile identity/version;
- configured total-ingress-failure mode;
- Drawbridge Service Address/ingress identity;
- Front Distributor identity;
- selected Gateway;
- fallback target and deterministic selection reason/order where applicable;
- prior Gateway;
- placement reason;
- Gateway eligibility/health state;
- Front Distributor health transition;
- drain start/completion;
- transport interruption;
- resume attempt/result;
- logical Device Session preserved/replaced;
- virtual identity/address preserved/changed;
- data-plane restoration time.

A Gateway Placement Profile or Front Distributor placement Record is not an authorization Record. Gateway authorization/enforcement remains independently attributable. Records must also make cross-profile selection attempts or unexpected direct-ingress attempts visible as security-relevant events.

Records must preserve enforcement-layer truth separately: Agent `TUNNEL` is not Gateway `ALLOW_FORWARD`; Gateway `ALLOW_FORWARD` is not proof that the enterprise firewall or destination application accepted the flow. When Agent and Gateway observations or policy generations disagree, preserve both producer-attributed facts rather than rewriting one side to match the other.
