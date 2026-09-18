# Client Sensor and DNS Policy

## 1. Client Sensor

Every Drawbridge endpoint should include a first-class sensor modeled conceptually after the Pathfinder UDM-Pro sensor.

The sensor is not a separate security product. It is the endpoint observation source for Drawbridge Records and policy verification.

It should observe, subject to platform capability and site policy:

- outbound connection attempts;
- inbound connection attempts where relevant;
- source/destination IP and port;
- protocol;
- process/application identity;
- user identity;
- DNS questions;
- DNS answers;
- DNS resolver used;
- interface/network in use;
- route selection;
- Drawbridge tunnel/direct decision;
- policy decision;
- session state;
- trusted/untrusted network state;
- network changes;
- location observations when enabled.

The sensor should preserve:

```text
what the endpoint observed
what DNS returned at that time
what Drawbridge decided
what transport path was selected
what the Edge later observed
```

## 2. DNS is a Drawbridge policy function

DNS handling must be explicitly controlled by Drawbridge policy.

The endpoint operating system must not be allowed to make uncontrolled DNS decisions that can bypass Drawbridge policy.

Drawbridge should support multiple DNS modes because different device classes have different needs.

## 3. DNS modes

### 3.1 Local DNS / local filtering

Use the DNS resolver supplied by the current local network.

Example use case:

- ordinary remote worker;
- home router or approved local filtering;
- public Internet traffic should remain local;
- Drawbridge only carries explicitly protected resources.

Example:

```text
public Internet DNS:
  local resolver

*.xyz.local.domain:
  Drawbridge/private resolver
```

Drawbridge still observes and records the query/answer where platform policy permits.

### 3.2 Protected namespace only

Only approved private namespaces are handled by Drawbridge.

Example:

```text
*.xyz.local.domain
*.cad.county.gov
*.rms.county.gov
```

These queries are sent to configured Drawbridge/enterprise DNS resolvers.

All other DNS is sent to the resolver associated with the local connection.

This mode is well suited to split-tunnel deployments.

### 3.3 Forced Drawbridge DNS

All DNS is sent through Drawbridge.

Use cases:

- organization requires centralized DNS filtering;
- endpoint must not use local DNS;
- organization requires consistent policy/logging;
- local network DNS is considered untrusted.

Drawbridge may forward to:

- enterprise DNS;
- approved filtering resolver;
- Drawbridge-hosted filtering service;
- customer-defined upstream resolvers.

### 3.4 Hybrid policy DNS

Different namespaces or applications may use different resolvers.

Example:

```text
*.xyz.local.domain
  -> enterprise DNS over Drawbridge

*.vendor.example
  -> approved filtering resolver

everything else
  -> local connection resolver
```

The policy must remain deterministic and explainable.

## 4. DNS decision record

Each DNS decision should be recordable as:

```text
device:
  MDT-014

user:
  DOMAIN\jwood

process:
  cad.exe

query:
  cad.xyz.local.domain

type:
  A

dns_policy:
  PROTECTED_NAMESPACE

resolver_selected:
  10.20.1.5

resolver_path:
  DRAWBRIDGE

reason:
  matched *.xyz.local.domain

policy_version:
  DB-POL-000184
```

Then the response:

```text
answer:
  cad.xyz.local.domain -> 10.40.20.15

ttl:
  300

observed_at:
  <timestamp>
```

And a later connection record can link:

```text
10.250.14.37 -> 10.40.20.15:443

name_at_time:
  cad.xyz.local.domain
```

## 5. FQDN enforcement

FQDN policy must be based on observed DNS state, not on permanent conversion of a hostname into a static IP allow-list.

Drawbridge should:

- observe the DNS answer;
- associate the answer with the requesting endpoint/process where practical;
- respect TTL and refresh;
- age stale bindings out;
- track CNAME chains;
- preserve the resolution-at-time record;
- reject unexplained destination addresses that are outside the active binding when policy requires strict FQDN enforcement.

Example:

```text
Policy:
  allow cad.xyz.local.domain:443

Observed:
  cad.xyz.local.domain -> 10.40.20.15
  TTL 300

Temporary authorized binding:
  10.40.20.15:443
  until refresh/expiry
```

## 6. Alternate DNS protocols

Drawbridge must account for applications that attempt to bypass system DNS using:

- DNS over HTTPS (DoH);
- DNS over TLS (DoT);
- hard-coded DNS servers;
- application-embedded resolvers.

Policy choices should include:

- allow;
- direct;
- tunnel;
- block;
- require approved resolver.

The endpoint sensor should record attempts where technically possible.

A site requiring strict DNS control may block unauthorized DoH/DoT or force them through approved services.

## 7. DNS leak prevention

When a namespace is marked protected, its query must not be sent to an untrusted local resolver unless the policy explicitly allows it.

Example:

```text
*.xyz.local.domain
  -> NEVER local DNS
  -> enterprise resolver only
```

This protects internal names from leakage and prevents false local answers from being trusted.

## 8. Local DNS dependency

Drawbridge should not assume that local DNS is always bad.

For many split-tunnel deployments, the simplest and most reliable policy is:

```text
public DNS:
  local connection resolver

protected enterprise names:
  Drawbridge/private resolver
```

The administrator chooses the behavior.

## 9. DNS and trusted networks

On a verified trusted enterprise network, DNS policy may transition to native enterprise behavior.

Example:

```text
trusted network:
  system/enterprise DNS

untrusted network:
  protected namespace -> Drawbridge DNS
  public namespace    -> local or Drawbridge according to policy
```

The transition and reason must be recorded.

## 10. Sensor and routing correlation

The client sensor should make it possible to reconstruct:

```text
DNS query
  ->
DNS answer
  ->
FQDN policy match
  ->
route decision
  ->
tunnel/direct decision
  ->
connection attempt
  ->
Edge observation
```

This is a core troubleshooting and forensic requirement.

## 11. Privacy and scope

The sensor must be policy-controlled.

Organizations should be able to configure which telemetry classes are collected and retained.

Drawbridge should not collect payload content merely because it can observe connection metadata.

## 12. v1 requirement

For v1, the minimum sensor/DNS target should be:

- connection attempts;
- DNS query/answer correlation;
- process attribution where Windows allows it;
- route/tunnel decision;
- resolver selection;
- protected-namespace routing;
- local versus Drawbridge DNS policy;
- strict handling of unauthorized alternate DNS where configured;
- Records correlation.
