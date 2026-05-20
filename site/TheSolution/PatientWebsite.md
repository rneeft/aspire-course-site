---
title: Patient Website
layout: home
parent: The Solution
nav_order: 1.5
---

# Patient Website

The **PatientWebsite** is the only service with a frontend. It runs on `http://localhost:8080`.

It is a server-rendered Razor Pages application that allows patients to:

1. **Register** — create their account (BSN + first name + last name)
2. **Sign in** — authenticate and receive a session cookie
3. **Manage consent** — approve or decline access for each healthcare company that has a dossier for them

## Patient registration

Navigate to `http://localhost:8080/Register`. Enter your BSN and name. On submission the website:

1. Creates a `BsnPseudoniem` record (BSN → GUID mapping)
2. Creates a `Patient` record (first name, last name, pseudoniem)

Both records are written directly to the shared database. The raw BSN is stored **only** in the `BsnPseudoniem` table.

Test BSN you can use: **`123456780`** (this is the default value in the Bruno environment).

{: .warning }
> Never use a real BSN — all BSNs in this workshop are fictional.

## Sign in

Navigate to `http://localhost:8080/SignIn`. Enter the BSN and last name. The website calls **`POST /auth/login`** on IdentityApi and receives a JWT. That JWT is converted into a **cookie-based session** so the patient stays logged in across page loads without the browser ever handling the raw token.

## Consent dashboard

After signing in, the home page (`/`) shows:

- The patient's name
- A list of all healthcare companies that have created a dossier for this patient
- **Approve** / **Decline** buttons for each company

Clicking **Approve** sets `Patient.GavePermission = true` in the database. The next time DossierApi calls `GET /dossier/{bsn}/permission` for that company, it will return `gavePermission: true`.

## Try the full flow

1. Register a patient at `http://localhost:8080/Register`
2. In Bruno, run `Dossier/RegisterCompany` then `Identity/GetToken` then `Dossier/CreateDossier`
3. Check `Dossier/CheckPatientPermission` — `gavePermission` should be `403 - Forbidden`
4. Sign in to the patient website and click **Approve**
5. Check `Dossier/CheckPatientPermission` again — `gavePermission` should now be `202 - OK`

Now that you understand the full solution, let's improve the local development experience by adding [.NET Aspire](../Aspire/).