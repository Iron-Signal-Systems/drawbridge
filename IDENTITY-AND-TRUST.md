# Identity and Trust

## 1. Principle

Drawbridge separates:

- device identity;
- user identity;
- network state;
- resource authorization;
- endpoint operating-system/application authentication.

Network connectivity does not imply application authorization.

## 2. Domain-managed mode

Domain-managed Windows is the v1 gold-path deployment.

The Domain Agent runs under a narrowly scoped gMSA. Device identity, Agent service identity, and User identity remain separate. The gMSA is not a substitute for Device identity, User authorization, or Resource policy.

### Device enrollment

The administrator configures one or more Mobility OUs.

Example:

```text
OU=Mobility,OU=Computers,DC=county,DC=gov
```

The Controller synchronizes those OUs and descendants according to policy.

Internally, Drawbridge should anchor devices using stable directory identifiers rather than distinguished names.

Recommended identity references:

- `objectGUID` for stable object identity;
- `objectSid` where security identity is required;
- current DN/OU as administrative location.

Moving a computer out of an authorized Mobility OU removes its device authorization according to policy.

### Device PKI

Domain PKI supplies the machine credential.

Recommended properties:

- dedicated Drawbridge/Mobility computer certificate template;
- client-authentication purpose;
- private key non-exportable where practical;
- TPM-backed key where practical;
- automatic enrollment;
- chain and revocation validation;
- explicit identity mapping to the synchronized AD computer object.

The certificate proves possession of the configured machine credential.

Certificate validity alone does not establish current Drawbridge authorization. Enrollment/revocation state and policy are evaluated independently.

AD OU state may contribute to current authorization while the domain is trusted.

### Bootstrap/device connection

The machine certificate may initiate a restricted pre-logon connection.

Typical allowed services:

- enterprise DNS;
- domain controllers as explicitly required;
- Kerberos;
- LDAP/LDAPS as explicitly required;
- SYSVOL/NETLOGON as required;
- NTP;
- PKI/CRL/OCSP;
- Drawbridge Controller/Gateway services;
- approved endpoint-management services.

Everything else remains denied until policy grants it.

### User authorization

Users are authorized through configured AD security groups.

Example:

```text
GG-Drawbridge-Users
GG-Drawbridge-Patrol
GG-Drawbridge-IT
```

AD group membership is an input to Drawbridge policy; it is not itself the policy.

The user's successful Windows/domain identity may be used without requiring a redundant username/password prompt where security policy allows.

NPS/RADIUS may be used where desired.

## 3. Shared-services/non-domain mode

Shared-services devices are not members of the hosting organization's AD domain and Drawbridge must not pretend they are.

### Device identity

Use a Drawbridge/shared-services PKI.

Preferred enrollment:

1. Agent generates the Device private key locally;
2. preferably TPM-backed and non-exportable;
3. Agent generates a Device CSR;
4. administrator approves device enrollment;
5. issuing CA signs device certificate;
6. endpoint stores certificate/private key locally.

The shared-services Root CA certificate is trusted by participating endpoints as required.

The private Root CA key is never distributed.

### Device certificate authorization

A valid certificate alone is insufficient.

Authorization should require:

```text
valid certificate
AND
approved issuing hierarchy
AND
correct EKU/profile
AND
certificate not revoked
AND
device enrollment still active
AND
tenant/agency policy permits connection
```

### User authentication

Drawbridge must not authenticate a foreign workstation against the hosting agency's AD merely because the service is hosted there.

Supported user authorities should include:

- OIDC/SAML federation to the user's home agency;
- RADIUS/NPS operated by the user's home agency;
- Drawbridge-local identity with strong MFA/FIDO2/passkey support;
- future compatible identity providers.

### Application authentication

Drawbridge answers:

> May this authenticated user/device establish network connectivity to this service?

Drawbridge does not automatically answer:

> Is this user authorized inside the destination application?

CAD/RMS/GIS/other applications may still require their own authentication.

## 4. Inbound management

For shared-service endpoints:

```text
default inbound management = DENY
```

Enabling remote administration requires:

- explicit Drawbridge policy;
- explicit allowed source;
- explicit allowed protocol/port;
- local endpoint authorization;
- normal Windows/local authentication.

Drawbridge must not silently create local accounts or grant local administrator rights.

The network path and endpoint login authority are separate controls.

## 5. Tenant model

Agency/tenant is a first-class object.

Each tenant should have independently scoped:

- devices;
- user mappings;
- certificate enrollment;
- resource authorization;
- administrators;
- Records visibility;
- retention where configurable;
- shared-service relationships.

Cross-tenant access requires explicit policy.

## 6. Device revocation and recovery

Any authorized in-scope Revocation Operator may immediately revoke a Device. Active Device Sessions terminate and new/resumed sessions are denied without waiting for AD or normal PKI publication.

A lost/stolen credential is never simply unrevoked. Recovery requires hands-on IT verification, new key/CSR, full Device certificate re-issue, renewed enrollment, and validation. The old credential remains permanently revoked.

## 7. Administrative authority

Drawbridge follows a DNP-style scoped Authority Grant model rather than simple role-name checks. Holding authority and exercising it are separate facts; delegation cannot exceed the delegator's authority.

## 8. Service identities

Gateway, Controller, Connector, and other Windows service hosts use host-specific gMSA sets, with function separation where permissions differ. Domain identity does not imply domain administrative authority.

## 9. Full domain compromise

Full AD compromise is collapse of that domain's trust boundary. Drawbridge does not claim continued trust in domain Users, computer accounts, gMSAs, Kerberos, AD groups, GPO, domain-derived administrators, or dependent domain-joined Windows hosts.

## 10. PKI trust failure

Once an issuing PKI authority is declared compromised, Drawbridge denies all new trust establishment through it, including new Device connections, fresh reconnects, initialization/bootstrap, enrollment/renewal, and affected infrastructure initialization.

Revoked security credentials are replaced, not reactivated.

## 11. Authorization equation

A useful baseline:

```text
device authorization
    INTERSECT
user authorization
    INTERSECT
tenant authorization
    INTERSECT
resource policy
    INTERSECT
current network state
    =
effective access
```

A broader grant in one dimension must not silently override a restriction in another.


See [THREAT-MODEL.md](THREAT-MODEL.md) and [RECOVERY-STORE.md](RECOVERY-STORE.md).
