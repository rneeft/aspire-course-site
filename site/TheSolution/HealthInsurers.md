---
title: Pseudoniem API
layout: home
parent: The Solution
nav_order: 1.3
---

# Pseudoniem API

The **PseudoniemApi** translates a patient's real BSN into a stable, pseudonymous GUID. This is the cornerstone of the platform's privacy design.

## Why pseudonymise?

A BSN is classified as sensitive personal data under Dutch law. If the DossierApi stored raw BSNs in dossier records, a database leak would immediately expose patient identities. Instead:

1. The BSN is passed to PseudoniemApi **once** at dossier creation time.
2. PseudoniemApi returns a deterministic GUID (a *pseudoniem*).
3. All subsequent operations use only the pseudoniem GUID.

The BSN-to-GUID mapping is stored in a single table (`BsnPseudoniem`) that only PseudoniemApi ever writes to.

## The endpoint

```
GET /pseudoniem/{bsn}
```

- Returns the existing pseudoniem if the BSN has been seen before.
- Creates and stores a new pseudoniem GUID on first call.
- Requires an `Internal` role JWT in the `Authorization: Bearer` header.

## Internal-only access

The endpoint is protected by the `Internal` authorization policy. Only a caller that holds a valid `Internal`-role token (issued by `POST /auth/token/internal`) will receive a `200 OK`. Any other token — or no token at all — gets a `403 Forbidden`.

This means PseudoniemApi cannot be called directly by a patient or a healthcare company. Only a trusted internal service (DossierApi) can reach it.

## Try it in Bruno

In Bruno, open **Identity → GetInternalToken** and send the request to populate `internal_token`. Then open **Pseudoniem → GetPseudoniemByBsn** and send it. The folder is pre-configured to use `internal_token` as the Bearer token.

Try changing the token to `company_token` — you should get a `403 Forbidden`.

## Next: [Dossier API](DataFile)
