---
title: Identity API
layout: home
parent: The Solution
nav_order: 1.2
has_children: true
---

# Identity API

The **IdentityApi** is the authentication service. It issues [JSON Web Tokens (JWT)](JWT) to three different types of caller, each with a different role:

| Role | Endpoint | Who uses it |
|------|----------|-------------|
| `Patient` | `POST /auth/login` | A patient logs in with BSN + last name |
| `HealthcareCompany` | `POST /auth/token` | A company authenticates with its `CompanyId` |
| `Internal` | `POST /auth/token/internal` | A service calls another service (no user involved) |

All tokens are signed with HS256 using the shared `JwtSettings__SecretSigningKey`. Any service that holds this key can independently verify a token without calling IdentityApi again.

## Patient login — `POST /auth/login`

A patient provides their BSN and last name. The API looks up the patient record, verifies the last name matches, and returns a token.

In Bruno, open **Identity → PatientLogin** and send the request. The script automatically saves the token to the `patient_token` environment variable.

The token payload looks like this:

```json
{
  "sub": "a1b2c3d4-...",
  "role": "Patient",
  "iss": "Online-Toestemming-Workshop-IdentityApi",
  "aud": "Online-Toestemming-Workshop",
  "exp": 1234567890
}
```

{: .note }
> Notice that `sub` is the patient's **pseudoniem** (a GUID), not their BSN. The BSN never leaves IdentityApi.

## Company token — `POST /auth/token`

A healthcare company provides its `CompanyId` (received when registering). The API returns a token with role `HealthcareCompany` and `sub` set to the company name.

In Bruno, open **Identity → GetToken** and send the request. The `company_token` variable is set automatically.

## Internal token — `POST /auth/token/internal`

This endpoint requires no credentials. It issues a short-lived (15-minute) token with role `Internal`. Only services running inside the network should ever call this endpoint — it is not exposed externally.

DossierApi calls this endpoint automatically when it needs to contact PseudoniemApi. You can also try it manually via **Identity → GetInternalToken** in Bruno.

{: .warning }
> In this workshop the token endpoint has no guard. In production you would protect it with mTLS, network policy, or a client credential.

## Next: [Pseudoniem API](HealthInsurers)