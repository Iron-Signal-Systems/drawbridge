# Licensing Principles

## 1. Site licensing

Drawbridge should use site/deployment licensing rather than per-user or per-device seat licensing.

The commercial unit is the deployed site/environment, not the individual MDT.

## 2. Endpoint and user counts

Normal endpoint/user counts are not a packet-forwarding entitlement.

A site may have:

- device refresh overlap;
- staging MDTs;
- spares;
- temporary emergency devices;
- changing user counts;
- agency growth;

without licensing logic choosing which device stops working.

## 3. Site license includes

Target model:

- production Controller;
- supported production HA topology, including redundant Front Distributor capacity and Gateway capacity appropriate to the Site;
- normal DR/standby capability;
- test/staging;
- test Devices/Agents;
- unlimited managed endpoints within supported engineering capacity;
- unlimited normal users;
- AD integration;
- PKI integration;
- NPS/RADIUS;
- OIDC/SAML/federation where supported;
- trusted networks;
- split routing;
- FQDN routing;
- Records;
- reporting/search;
- API;
- software updates;
- normal support.

Exact commercial packaging is TBD.

## 4. Capacity is engineering, not entitlement

Drawbridge should expose:

- connected devices;
- active sessions;
- throughput;
- CPU;
- memory;
- transport health;
- virtual-address capacity;
- Records ingestion load.

When a site outgrows its Front Distributor/Gateway capacity or failure-domain design, add capacity because the infrastructure needs it—not because endpoint #201 crossed a licensing threshold. Licensing does not hard-code a two-Gateway topology.

## 5. Licensing and the data path

Hard requirement:

> Commercial licensing must never terminate an already-authorized session or stop forwarding traffic under the last valid security policy.

Licensing may affect:

- new commercial deployment creation;
- support eligibility;
- upgrades;
- expansion beyond contracted site scope;
- hosted services;
- additional production sites/clusters.

Licensing must not affect:

- current packet forwarding;
- established security policy enforcement;
- session continuity;
- device-management connectivity;
- public-safety production operation.

## 6. Restart behavior

An expired or unavailable commercial license service must not turn a healthy production environment into an outage after:

- patching;
- reboot;
- service restart;
- failover;
- Gateway replacement.

Operational state needed for continuity must be locally durable.

## 7. Commercial state versus security state

These are never equivalent:

```text
commercial subscription expired
!=
device certificate revoked
!=
user authorization revoked
!=
device declared stolen
```

Commercial problem:
- alert;
- administrative restriction where appropriate;
- production continuity.

Security problem:
- enforce revocation/quarantine immediately according to policy.

## 8. Test/DR must not be separately punished

A site license should include non-production test/staging and normal DR.

Charging separately for test encourages unsafe direct-to-production behavior and conflicts with Drawbridge's design principles.
