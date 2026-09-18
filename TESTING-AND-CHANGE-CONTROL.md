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
- Agent behavior;
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
- Gateway/Controller environment configuration;
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

## 6. Production promotion and verified execution

Promotion creates an immutable production event containing:

- old version;
- new version;
- promoter;
- reason;
- linked test run(s);
- validators;
- timestamp;
- resulting expected configuration hash.

The Controller distributes the promoted declarative artifact over the authenticated persistent control channel established by each managed component.

The target independently verifies target/version/hash/authorization, applies locally, verifies effective state, and returns a receipt distinguishing sent, received, validated, applied, verified, and failed.

After a production promotion/change, each affected Gateway/Controller creates an immediate Drawbridge Recovery Store checkpoint.

## 7. Rollback/revert

Production versions advance monotonically.

A rollback creates and tests a new higher version whose effective configuration intentionally restores prior behavior. Enforcement components do not simply activate an older production artifact.

The original versions remain historical facts.

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

Gateway/Controller and Agent upgrades follow the same model.

Before production:

- deploy to test Controller/Gateway;
- validate database migration;
- validate certificates;
- validate AD sync;
- validate NPS/RADIUS;
- validate policy;
- validate session migration;
- validate Records;
- validate failover;
- validate Agent compatibility;
- perform operational MDT testing where relevant.

## 10. Upgrade preflight

Production upgrade should be blocked on critical preflight failures such as:

- invalid/unknown schema state;
- PKI failure;
- insufficient disk;
- unhealthy HA peer;
- incompatible Gateway/Controller versions;
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


## 13. Control-channel negative testing

Tests must prove managed components reject artifacts targeted to another system, invalid hashes, unsupported schemas, stale/older production versions, retired protocol/cryptographic versions, undefined generic control operations, and duplicate object identifiers with conflicting content.

## 14. Recovery testing

Acceptance includes DRS checkpoint age, immediate checkpoint after production change, stored-hash verification, lineage/gap detection, recovery-object export, rebuild of a test Gateway/Controller from a verified recovery object, and proof that DRS administration does not depend on production AD.

A recovery test creates new history; it does not mutate historical DRS objects.
