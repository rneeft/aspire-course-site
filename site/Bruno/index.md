---
title: Bruno
layout: default
nav_order: 3
---

# Bruno

[Bruno](https://www.usebruno.com/) is an open-source, offline API client. Unlike Postman, collections are stored as plain text files in your repository — no cloud account required.

## Import the collection

1. Install Bruno from [usebruno.com](https://www.usebruno.com/).
2. Click **Open Collection**.
3. Navigate to `docs/Bruno` in this repository and open the folder.

The collection is organised into three folders matching the three APIs:

| Folder | Contents |
|--------|----------|
| **Identity** | GetToken, PatientLogin, GetInternalToken |
| **Pseudoniem** | GetPseudoniemByBsn |
| **Dossier** | RegisterCompany, CreateDossier, CheckPatientPermission, DeleteDossier |

## Set the environment

Switch to the **Local** environment using the dropdown in the top-right corner. The `Local` environment pre-sets:

| Variable | Default value | Set by |
|----------|--------------|--------|
| `bsn` | `123456780` | Pre-filled |
| `company_id` | *(empty)* | Set by RegisterCompany script |
| `company_name` | `Test Healthcare Company` | Pre-filled |
| `company_token` | *(empty)* | Set by GetToken script |
| `patient_token` | *(empty)* | Set by PatientLogin script |
| `internal_token` | *(empty)* | Set by GetInternalToken script |

Post-response scripts in each request automatically populate token variables, so you don't have to copy-paste them manually.

## Run the full test sequence

Open **Dossier → FullTestSequence** for a description of the recommended order. The quick version:

```
1. Dossier/RegisterCompany          → 201 Created  (sets company_id)
2. Identity/GetToken                 → 200 OK       (sets company_token)
3. Dossier/CreateDossier             → 201 Created
4. Dossier/CheckPatientPermission    → 200 OK, gavePermission: false
   --- patient logs in at http://localhost:8080 and approves ---
5. Dossier/CheckPatientPermission    → 200 OK, gavePermission: true
6. Dossier/DeleteDossier             → 204 No Content
```

## Bruno vs Postman

| Feature | Bruno | Postman |
|---------|-------|---------|
| Collection storage | Plain `.bru` text files in your repo | JSON file, cloud-synced |
| Account required | No | Yes (for sync/sharing) |
| Offline | Fully offline | Limited offline support |
| Scripting | JavaScript (pre/post-request) | JavaScript |
| Environment variables | Yes | Yes |
