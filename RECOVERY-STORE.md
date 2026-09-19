# Drawbridge Recovery Store

## Purpose

The Drawbridge Recovery Store (DRS) preserves independently verifiable known-good Drawbridge configuration outside the production Windows and Active Directory trust boundary.

It supports recovery after full domain compromise, Controller/Gateway compromise, destructive configuration error, loss of normal production trust, and rebuild from trusted media.

The DRS is deliberately small. It is not a Controller, Records server, policy engine, packet-processing system, or directory service.

## Recovery objects

Every production Gateway and Controller exports a complete normalized configuration recovery object:

- immediately after each production configuration promotion/change;
- at least every 12 hours even when no change occurred.

Objects are complete snapshots, not deltas, and include source identity/role, configuration version, effective policy/configuration identifiers, creation time, content hash, lineage reference, and required schema/version data.

Any production configuration required to reconstruct another Drawbridge component must either be deterministically regenerable from DRS-protected authoritative configuration or be preserved in DRS itself. Local-only recovery-critical configuration is not allowed to exist outside both of those paths.

## Secret exclusion

DRS preserves configuration, not reusable runtime secrets.

Recovery objects must not contain:

- private keys;
- passwords;
- bearer or refresh tokens;
- reusable MFA material;
- CA signing keys;
- production software-signing keys;
- recoverable gMSA secrets.

Recovery creates new host/service identities and new private credentials rather than restoring compromised secret material.

## Immutable from birth

A canonical DRS object is immutable from birth.

The producer constructs/hashes the complete object. DRS receives it into bounded staging. DRS independently validates/hashes it. A host-side Sealer creates a new canonical object, verifies the stored object, seals it, and returns a receipt for the exact object/hash.

There is no append, patch, update, or rewrite operation on a canonical object.

## Trust independence

The DRS must not be domain joined.

Storage integrity and recovery administration must not depend exclusively on the production AD domain.

A SAN-backed/virtual deployment is permitted only if the infrastructure capable of modifying/destroying DRS does not collapse back into the same trust boundary.

A small dedicated physical appliance is a strong option. Reliability and trust independence matter more than CPU performance.

## Preferred FreeBSD architecture

Use an extremely hardened FreeBSD system with ZFS, PF, VNET jails, minimal software, independent recovery authentication, and tightly controlled management networking.

Externally reachable functions live in separate VNET jails:

- Ingest Jail;
- Recovery Jail;
- optional Replication Jail.

A host-side Sealer has no network listener.

## Ingest Jail

Accepts authenticated outbound pushes from authorized Gateways/Controllers.

It may authenticate the producer, receive a complete object, validate framing/schema, calculate hash, place the completed object in bounded staging, and return bounded protocol status.

It has no mutable access to canonical recovery storage.

## Host-side Sealer

The Sealer is the narrow privileged boundary. It accepts narrow host-local requests, validates staged objects, independently checks hashes/metadata/lineage, creates and durably commits the canonical object, verifies it, seals it, and generates the receipt.

It provides no general command execution, filesystem browsing, remote administration, or production Drawbridge control.

## Recovery Jail

Provides independently authenticated, read-only access to sealed recovery objects from the dedicated recovery/management network.

It may list, verify, and export recovery points. It cannot modify canonical objects or accept production ingestion.

## Replication Jail

May send read-only copies of sealed objects to an independent secondary location or immutable cloud/backup target. Preferred flow is outbound from DRS.

## Storage and network boundaries

Separate OS state, staging, canonical objects, derived indexes/metadata, and replication state.

Canonical storage is not mounted writable into network-facing jails.

Management, ingestion, recovery, and replication planes are independently controlled with PF/VNET and, where appropriate, separate NICs/VLANs.

## Recovery process

1. isolate affected production systems;
2. identify the last defensible known-good DRS recovery point;
3. verify hash, lineage, and source metadata;
4. rebuild affected hosts from trusted media and an independently verified authorized software release;
5. create new host/service identities;
6. issue new credentials from a trusted PKI path;
7. restore the selected verified configuration;
8. run staging/self-review/validation;
9. create a new production configuration identity;
10. return the rebuilt component to service.

A historical DRS object is a recovery source, not live policy authority.


## Availability boundary

DRS is not a forwarding or live-authorization dependency. DRS unavailability increases recovery risk and must raise health/operational alerts, but it does not by itself terminate otherwise authorized production traffic.
