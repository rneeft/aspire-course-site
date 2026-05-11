---
title: Let's give it a go
layout: home
parent: The Solution
nav_order: 1.1
---

# Let's give it a go

Let's start the entire platform with a single command and verify it's working.

## 1. Create a `.env` file

The Docker Compose setup requires two environment variables. In the project root, create a file called `.env`:

```ini
SQL_PASSWORD=YourStr0ngPassword!
JWT_SECRET_KEY=ThisIsASecretKeyForJwtSigningPleaseChangeItInProduction
```

{: .note }
> `JWT_SECRET_KEY` must be at least 44 characters long. All services share this key, so they can all verify each other's tokens.

## 2. Start the application

Open a terminal in the project root and run:

```bash
docker compose up --build -d
```

Docker will:
1. Build images for all five services
2. Start SQL Server and wait until it is healthy
3. Run the **MigrationService** to create the database schema, then exit
4. Start **IdentityApi**, **PseudoniemApi**, **DossierApi**, and **PatientWebsite**


## 3. Import the Bruno collection

[Bruno](https://www.usebruno.com/) is a free open-source API client. The workshop provides a Bruno collection under `docs/Bruno/`.

1. Open Bruno.
2. Click **Open Collection**.
3. Select the `docs/Bruno` folder.
4. Switch to the **Local** environment (top-right dropdown).

The `Local` environment pre-fills variables such as `bsn` (`123456780`) and stores tokens automatically after each login request.

## 4. Run the full test sequence

Open the **Dossier → FullTestSequence** file in Bruno. It documents the complete happy path:

| # | Request | Expected result |
|---|---------|----------------|
| 1 | `Dossier/RegisterCompany` | `201 Created`, sets `company_id` |
| 2 | `Identity/GetToken` | `200 OK`, sets `company_token` |
| 3 | `Dossier/CreateDossier` | `201 Created` |
| 4 | `Dossier/CheckPatientPermission` | `200 OK`, `GavePermission: false` |
| 5 | Patient registers & approves consent on PatientWebsite (`http://localhost:8080`) | — |
| 6 | `Dossier/CheckPatientPermission` (again) | `200 OK`, `GavePermission: true` |
| 7 | `Dossier/DeleteDossier` | `204 No Content` |

Run the requests in order. In the next pages you'll understand exactly what each service is doing.

You can also run then all by right clicking the 'OnlineToestemming' folder and click on Run. Leave the defaults as is and click on 'Run 9 Requests'. 

Lets explore what we have running by looking at the [Identity Api](IdentityApi).