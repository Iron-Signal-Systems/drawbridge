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
- Policy decision
- Session established
- Session suspended
- Session resumed
- Session migrated
- Session terminated
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

Location is a first-class observation but is unusually sensitive.

Possible sources:

- GNSS;
- Windows Location Service;
- Wi-Fi positioning;
- cellular positioning;
- trusted-network/site classification;
- administrator-defined site identity.

Each record should include:

- source;
- timestamp;
- accuracy where available;
- age/freshness where available;
- raw observation appropriate to configured privacy level.

Physical location and network-derived location must not be silently conflated.

## 6. Location controls

Site policy should support modes such as:

- disabled;
- network-location only;
- coarse physical location;
- precise physical location.

Access to location records should itself be auditable.

Retention may differ from general connection metadata.

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
- `tenant_id`
- `test_run_id`

## 9. Append-oriented history

Historical observations should not be mutated when new intelligence appears.

If Drawbridge later concludes that an old destination was malicious, create a new assessment linked to the original record.

## 10. Integrity

Future target:

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
