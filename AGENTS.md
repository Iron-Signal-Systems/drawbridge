# Drawbridge Agent and Contributor Rules

## Purpose

This file defines how contributors, coding agents, automation, and review agents
work within the Drawbridge repository.

It is a behavioral and engineering contract. It does not replace the governing
architecture, trust, policy, Records, routing, testing, licensing, operations,
or roadmap documents.

Before making a material change, read the relevant current documents:

- `README.md`
- `TERMINOLOGY.md`
- `ARCHITECTURE.md`
- `HA-AND-SESSION-CONTINUITY.md`
- `THREAT-MODEL.md`
- `IDENTITY-AND-TRUST.md`
- `POLICY-AND-STATE.md`
- `ROUTING-AND-DEPLOYMENT.md`
- `AGENT-SENSOR-AND-DNS.md`
- `RECORDS.md`
- `RECOVERY-STORE.md`
- `TESTING-AND-CHANGE-CONTROL.md`
- `LICENSING.md`
- `OPERATIONS.md`
- `ROADMAP.md`
- `PHASE-0-REVIEW.md`
- `OPEN-QUESTIONS.md`

If implementation and documentation disagree, do not silently choose whichever
is easier. Identify and resolve the conflict deliberately.

---

## Canonical Terminology

`TERMINOLOGY.md` is normative for architecture terminology.

Use the exact actor or component name when identity, authority, routing, policy,
or Records attribution matters. In particular:

```text
User
Device
MDT
Drawbridge Agent
Domain Agent
Shared-Service Agent
Drawbridge Service Address
Drawbridge Front Distributor
Drawbridge Gateway
Drawbridge Controller
Drawbridge Recovery Store
Service Connector
Device Session
User Session
Transport Path
Resource
Site
Tenant / Agency
```

Do not use `Client` or `Edge` as ambiguous standalone Drawbridge architecture
terms. Historical references may use those names, and protocol-standard phrases
such as X.509 client authentication remain valid when they have a precise
technical meaning.

Do not substitute Device for User, Agent for Device, or Gateway for Device.
When a statement depends on a security boundary, name the exact actor.

---

## Governing Principles

> **Connectivity is not trust.**

> **The device establishes the transport; user authorization controls user access.**

> **A connected tunnel is not proof that required services are functional.**

> **Commercial licensing must never interrupt authorized production connectivity.**

> **Production advances through attributable, documented, tested versions.**

> **The exact configuration tested is the configuration promoted.**

> **Test/staging is a first-class feature.**

> **Records are a first-class subsystem, not a reporting add-on.**

> **Observed endpoint facts, DNS answers, policy decisions, Gateway observations,
> and later conclusions are different facts.**

> **Canonical Records and recovery objects are immutable from birth. Later
> facts create new canonical objects; they do not mutate old ones.**

> **Shared-service connectivity does not imply domain trust or endpoint
> administrative authority.**

> **Drawbridge should make secure change safer than doing nothing.**

---

## Product Boundary

Drawbridge is a separate Iron Signal Systems product.

It may integrate with Stronghold, but Stronghold is not required.

Drawbridge owns:

```text
device mobility identity
persistent remote transport
stable service ingress and Front Distributor placement
device/bootstrap connectivity
user access authorization
trusted-network transitions
Agent traffic selection
Drawbridge-specific resource policy
endpoint and Gateway observation
DNS policy related to Drawbridge access
session continuity
Records
test/staging
configuration promotion/rollback
```

Drawbridge is not automatically:

```text
the enterprise firewall
the endpoint Windows authentication authority
the destination application authorization authority
an EDR
a DLP platform
a CASB
a SWG
a SIEM replacement
```

Do not pull unrelated product responsibilities into Drawbridge merely because
the endpoint agent can observe traffic.

---

## Identity and Trust Rules

Keep these concepts distinct:

```text
device identity
user identity
tenant/agency identity
network state
resource authorization
endpoint OS authentication
destination application authentication
```

### Agent implementations

Domain Agent and Shared-Service Agent are distinct top-level implementations/contracts. Do not implement this trust distinction as scattered runtime mode checks.

Agents do not trust, administer, authenticate to, or directly control peer Agents.

### Domain-managed devices

Preferred model:

```text
configured Mobility OU
+
domain computer certificate
+
configured AD user/group authorization
```

The machine certificate may establish restricted device/bootstrap connectivity.
It does not automatically grant full interactive-user remote access.

### Shared-service devices

Do not authenticate a foreign/shared-service workstation against the hosting
agency domain merely because the hosting agency provides the application.

Use explicit Drawbridge/shared-services PKI and tenant authorization.

Inbound management defaults to deny and requires explicit Drawbridge policy plus
normal endpoint authentication.

### Stable identity

Prefer durable identity anchors over mutable names, distinguished names, current
IP addresses, or current external network addresses.

---

## Privilege Rules

Keep privileged endpoint functions narrowly bounded.

A privileged service/driver may exist for:

```text
virtual networking
WFP enforcement
protected key use
traffic classification
tunnel plumbing
approved sensor functions
```

Do not turn it into:

```text
generic command execution
PowerShell host
arbitrary filesystem reader
remote-administration backdoor
credential broker
generic registry administration service
unbounded LocalSystem RPC endpoint
```

Drawbridge must never silently create a local administrator or support account.

---

## Routing Rules

Routing is a deployment choice.

Keep endpoint traffic selection separate from enterprise handoff.

Supported directions include:

```text
split/selective tunnel
full tunnel
trusted-network bypass
routed DMZ handoff
optional NAT compatibility
internal service proxy/connector
```

If the customer wants the existing firewall to own enterprise routing and
segmentation, do not force a proxy architecture.

Prefer preservation of the Drawbridge virtual Device address when routed
handoff is possible.

---

## DNS Rules

DNS is explicit Drawbridge policy.

Supported directions include:

```text
local DNS
protected-namespace Drawbridge DNS
forced Drawbridge DNS
hybrid resolver policy
```

Protected namespaces must not silently leak to an untrusted local resolver.

FQDN policy must remain tied to observed DNS state, including TTL/refresh
behavior.

Do not convert a hostname into a permanent broad IP allow-list.

Account for CNAMEs, A/AAAA answers, DoH, DoT, hard-coded resolvers, stale
bindings, private namespaces, and trusted-network transitions.

---

## Agent Sensor and Records Rules

Every Drawbridge endpoint has a first-class sensor informed by the Pathfinder
UDM-Pro sensor model.

Preserve distinctions between:

```text
connection attempt
DNS question
DNS answer
process/user context
route decision
tunnel/direct decision
policy decision
Agent action
Gateway observation
later assessment
```

Denied traffic remains operationally and forensically important.

Canonical Records are complete write-once objects and immutable from birth.

Do not append to, patch, or rewrite a canonical Record. Later facts create new canonical objects linked to prior facts.

Agent and Gateway observations remain independently attributable.

A later success does not erase an earlier failure.

---

## Common Truth Separations

Never collapse:

```text
transport connected                    != service functional
device authenticated                   != user authorized
user authorized                        != application authorized
certificate valid                      != device permitted
commercial license expired             != device revoked
virtual IP                             != physical/external IP
current IP                             != durable identity
SSID                                   != trusted network
trusted network                        != allow any
FQDN                                   != permanent IP allow-list
DNS answer now                         != historical DNS answer
tunnel decision                        != firewall allow decision
Drawbridge allow                       != application allow
outbound access                        != inbound management
shared-service access                  != domain trust
shared-service access                  != local admin rights
Agent said sent                       != Gateway received
Front Distributor placement              != Gateway authorization
Gateway received                          != Gateway forwarded
Gateway forwarded                         != destination accepted
connection denied                      != unobserved
unknown                                != false
test config                            != production config
manually recreated config              != tested artifact
rollback                               != activating an old production version
recovery source                         != live policy authority
valid historical artifact              != current authorized artifact
service running                        != system healthy
```

---

## Change-Control Rules

Every production-affecting change is versioned and contains:

```text
parent version
new version/commit ID
author
timestamp
mandatory human reason/comment
exact diff
affected objects
test status
validation status
promotion status
```

Normal workflow:

```text
candidate change
    ->
commit
    ->
deploy exact candidate to test
    ->
validate
    ->
promote exact candidate
```

Do not manually recreate a tested change in production.

Emergency changes are allowed when operational restoration requires them, but
still require a named administrator, mandatory reason, exact diff, emergency
classification, post-change validation, and review.

Rollback/revert creates a new higher production version that intentionally restores prior behavior; it does not activate an older production artifact or erase history.

Managed components accept declarative artifacts through the bounded control protocol and independently verify target, version, hash, authorization, and effective state.

---

## Test/Staging Rules

Test/staging is first-class and part of the normal site model.

Use it to validate:

```text
policy
routing
DNS
PKI
AD
NPS/RADIUS
trusted networks
mobility
Agent upgrades
Controller/Gateway upgrades
Records
Front Distributor failover
Gateway failover
MDT workflows
```

The exact artifact tested is the artifact promoted.

Preserve failed and partial test runs as engineering history.

---

## Licensing Rules

Commercial licensing is outside the live forwarding decision.

The intended model is site/deployment licensing, not user/device seat licensing.

Never implement behavior where subscription expiry, licensing service failure,
capacity counters, patching, reboot, or failover causes otherwise authorized
production traffic to stop.

Commercial state and security revocation are separate.

---

## Operational Health Rules

Do not define health as merely:

```text
service running
port listening
Agent reports connected
```

Expose meaningful health for:

```text
control plane
device authentication
user authorization
policy engine
data plane
DNS
PKI
directory integration
Records
HA
resource reachability
```

Detect latent future failures before reboot, patching, failover, or upgrade when
reasonably possible.

---

## Windows Implementation Rules

The first serious Device/Agent target is Windows.

Prefer supported Windows-native facilities for production behavior, including
WFP, Windows networking APIs, Windows certificate stores/CNG, service APIs, and
TPM-backed keys where appropriate.

Do not shell out to PowerShell or administrative command-line tools from
production Agent code when a stable native API is the correct interface.

PowerShell remains appropriate for lab orchestration, deployment examples,
acceptance testing, and diagnostic collection.

---

## Go Coding Rules

Production Drawbridge code should favor Go unless a platform boundary requires
another implementation.

Use `switch` for discrete states/cases.

Use `if` for simple guards, ranges, boolean conditions, and compound
predicates.

Within a coherent file/section, keep functions and types alphabetized where that
does not damage lifecycle/readability ordering.

Avoid semantic nulls. Use explicit states once schema contracts are defined.

Errors should preserve operation, component, identity/session/resource where
safe, expected state, and actual state.

Do not swallow errors merely to keep the Agent appearing connected.

---

## Cryptography Rules

Do not invent cryptographic protocols.

Separate key purposes for device identity, shared-service identity, Gateway,
Controller, configuration signing, Records signing/checkpointing, and
administrative identity.

A valid TLS connection is not itself authorization.

A certificate chaining to a trusted CA is not by itself sufficient Device
authorization.

If an issuing PKI authority is declared compromised, deny all new trust establishment through it, including new connections, fresh reconnects, initialization/bootstrap, enrollment/renewal, and affected infrastructure initialization.

Define trust root, identity, purpose, enrollment, rotation, revocation, expiry,
and failure behavior before depending on a credential.

Never log private keys, passwords, bearer tokens, or reusable authentication
secrets.

---

## Software Update Trust Rules

Production updates must bind an exact GitHub source commit pin to a strong artifact hash and signed ISS release manifest that also identifies component purpose, platform, and version.

Do not use floating Git references such as `main`, `HEAD`, or `latest` as production update identities.

GitHub is provenance, not the sole runtime software trust root. Verification of an approved artifact must not require live GitHub access.

The normal Controller configuration channel must not become a generic binary-installation or arbitrary-code-execution path.

Runtime Drawbridge components must not possess production software-signing private keys.

A historically valid release may still be currently revoked or below the minimum accepted version.

---

## Security Rules

Do not:

```text
add a vendor backdoor
add an undocumented support account
add a generic remote shell
grant endpoint administration because network access exists
disable certificate validation in production
silently fail open
silently broaden a protected namespace
silently broaden a resource ACL
silently switch protected DNS to local DNS
silently bypass the tunnel after policy failure
allow licensing state to stop public-safety forwarding
rewrite, append to, or patch canonical Records
give a network-facing DRS jail mutable access to canonical recovery storage
place reusable runtime secrets or signing private keys in DRS recovery objects
treat full domain compromise as though domain-managed identities remain trustworthy
autonomously apply a recommended production policy change
```

Diagnostics must preserve normal trust boundaries.

---

## Testing Rules

Every production code change requires tests at the narrowest meaningful level.

For relevant Go changes, use at least:

```text
gofmt
go test ./...
go vet ./...
git diff --check
```

Platform-sensitive behavior requires platform-representative testing.

Mobility validation should eventually cover Wi-Fi/Ethernet/cellular transitions,
temporary loss of connectivity, sleep/resume, NAT rebinding, Front Distributor failure,
Gateway failover, and Controller unavailability.

A later success does not erase an earlier failed test.

---

## Documentation Rules

Documentation is part of the product contract.

Update governing documents when a change affects architecture, trust boundary,
identity, routing, DNS, policy meaning, state machine, Records, PKI, HA,
licensing behavior, test/promotion workflow, operator workflow, or supported
platform behavior.

Do not turn a design target into a measured production claim.

---

## Repository Hygiene

Do not commit private keys, passwords, tokens, real customer identity data, real
precise-location histories, production packet captures, generated Records
stores, machine-specific state, temporary backups, or unrelated experiments.

Before a proposed commit, review:

```text
git status --short
git diff --check
git diff --stat
git diff
```

and staged changes explicitly.

---

## GitHub and Change-Control Rules

Agents and automation must not perform repository writes without explicit user
authorization for that action.

Without explicit approval, do not:

```text
commit
push
merge
create or modify pull requests
change remote branches/tags
change rulesets or branch protection
change repository settings
create releases
delete GitHub files
```

A prior approval for one action is not blanket approval for future actions.

---

## Narrow-Fix Discipline

For a narrow defect:

1. identify the violated contract;
2. fix the smallest correct ownership/logic boundary;
3. add a regression test;
4. avoid unrelated refactors;
5. update governing documentation;
6. run relevant validation.

Do not solve a local defect by broadening access or weakening identity/policy.

---

## Final Engineering Rule

Drawbridge should remain understandable to the administrator operating it during
an outage and invisible enough to the field user that normal mobility requires
no networking expertise.

Prefer explicit identity, deterministic policy, bounded privilege, stable
mobility, observable DNS, traceable routing, immutable-from-birth canonical Records and recovery objects,
production-equivalent testing, attributable change, clear failures, and safe
continuity over hidden magic, opaque precedence, silent fallback, license-driven
outages, unreviewed production edits, or unverifiable history.


---

## Compromise and Recovery Rules

The normative compromise model is THREAT-MODEL.md.

Gateway, Controller, and Connector Windows service identities are host scoped; separate functions use separate gMSAs where permissions differ.

A lost/stolen Device may be revoked immediately by one in-scope Revocation Operator. Return to service requires hands-on verification and a new Device credential.

DRS is outside production AD trust. Gateways/Controllers push complete recovery objects after each production change and at least every 12 hours. Network-facing DRS VNET jails never receive mutable canonical-store access.

A full domain compromise is a domain trust collapse. Recovery uses independently protected trust and verified configuration.


## Threat-Model Baseline Rules

Issue #2 is closed at the design-principle level. Implementation work must preserve THREAT-MODEL.md rather than silently redefining its compromise, availability, telemetry, supply-chain, or accepted-risk boundaries.

Location is a normal first-class Records observation where platform capability permits it; preserve source, freshness, and accuracy rather than treating location as infallible truth.


## HA and Ingress Rules

Production Agents target the stable Drawbridge Service Address rather than named Gateways.

The Drawbridge Front Distributor is a narrow traffic-placement/health component. It does not grant Device/User/Tenant/Resource authorization and must not become a second Controller.

Gateways independently validate/enforce the Drawbridge session and authorization state for which they are responsible.

Individual Front Distributor nodes should be disposable/reconstructable where practical.

Do not add a rarely exercised direct-to-Gateway emergency production path when the Front Distributor tier is unavailable. Additional resilience must preserve the same Service Address -> Front Distributor -> Gateway architecture.

Transport, Front Distributor, or Gateway failure does not by itself redefine Device identity, User authorization, or the logical Device Session.
