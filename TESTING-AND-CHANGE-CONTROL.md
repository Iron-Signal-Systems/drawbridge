# Testing and Change Control

## 1. Test/staging is first-class

A licensed Drawbridge site includes a safe, production-equivalent test/staging environment.

The purpose is to allow:

- Network IT;
- Systems/PKI;
- Security;
- PD Operations;
- 911 / Emergency Management;
- application owners;

to prove changes before production is affected.

Test must support the same core behavior as production:

- policy engine;
- PKI;
- AD integration;
- NPS/RADIUS;
- client behavior;
- trusted-network behavior;
- mobility;
- Records;
- routing;
- DNS behavior.

## 2. Exact-artifact promotion

The configuration tested must be the configuration promoted.

Forbidden workflow:

```text
make change in test
test it
manually recreate change in production
```

Required workflow:

```text
candidate commit
    ->
deploy exact commit to test
    ->
validate
    ->
promote exact commit to production
```

## 3. Git-like configuration history

Every production-affecting change must have:

- parent version;
- unique commit/version ID;
- author;
- timestamp;
- mandatory human comment/reason;
- exact diff;
- affected objects;
- test status;
- validation status;
- promotion status.

Production is not anonymously edited.

## 4. Mandatory comments

Changes requiring comments include:

- policy;
- routes;
- resources;
- FQDN definitions;
- trusted networks;
- AD OU/group mappings;
- NPS/RADIUS;
- PKI/CA configuration;
- DNS;
- tenant configuration;
- remote-management authorization;
- Records/retention policy;
- Edge/controller environment configuration;
- upgrades.

The interface should reject empty or meaningless comments.

## 5. Validation record

A candidate may have multiple validators.

Example:

```text
Candidate:
  DB-POL-000184

PD Operations:
  PASS
  CAD login
  dispatch workflow
  LTE -> Wi-Fi
  Wi-Fi -> LTE
  dead-zone recovery

911 / EMO:
  PASS
  dispatch workflow
  reconnect behavior
```

Validation produces durable Records linked to the candidate.

## 6. Production promotion

Promotion creates an immutable production event containing:

- old version;
- new version;
- promoter;
- reason;
- linked test run(s);
- validators;
- timestamp;
- resulting effective configuration hash.

## 7. Rollback/revert

History is append-only in spirit.

A rollback creates a new version that reverts a prior change. It does not erase the bad version.

## 8. Emergency changes

Emergency production changes are permitted because public-safety restoration can outweigh normal workflow.

Requirements:

- explicit emergency classification;
- mandatory reason;
- incident/reference number where available;
- named administrator;
- exact diff;
- optional/required expiry for temporary access;
- post-change validation;
- visible unresolved-review state until reviewed.

Temporary emergency changes should not silently become permanent.

## 9. Upgrade staging

Server/client upgrades follow the same model.

Before production:

- deploy to test Controller/Edge;
- validate database migration;
- validate certificates;
- validate AD sync;
- validate NPS/RADIUS;
- validate policy;
- validate session migration;
- validate Records;
- validate failover;
- validate client compatibility;
- perform operational MDT testing where relevant.

## 10. Upgrade preflight

Production upgrade should be blocked on critical preflight failures such as:

- invalid/unknown schema state;
- PKI failure;
- insufficient disk;
- unhealthy HA peer;
- incompatible Edge/Controller versions;
- failed database backup/checkpoint;
- unresolved critical configuration validation issue.

Commercial license state must not make production forwarding fail after reboot. See LICENSING.md.

## 11. Test-device workflow

Drawbridge should make replacement and test MDTs easy to enroll without consuming artificial seat counts.

Operators should be able to test:

- boot off-site;
- machine authentication;
- pre-logon/device connectivity;
- domain login;
- user authorization;
- CAD/RMS/GIS;
- remote management;
- LTE/Wi-Fi transitions;
- dead-zone recovery;
- trusted-network transition;
- Records completeness.

## 12. Confidence model

Test results should be visible to all authorized stakeholders so the system builds shared confidence rather than requiring each team to trust another team's assumptions.
