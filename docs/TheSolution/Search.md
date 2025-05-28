---
title: Searching
layout: home
parent: The Solution
nav_order: 1.5
---

# Searching for Health Insurance Status

Healthcare providers need to know the health insurance status of a patient to handle billing, apply discounts, or verify coverage. This information is essential for ensuring that services are billed correctly and that patients receive the benefits they are entitled to.

Querying the data is done with the Burgerservicenummer (Social security number) of the patient. A BSN (Burgerservicenummer) is a unique personal identification number issued to every resident of the Netherlands. Companies are normally not allowed to use the BSN, however some parties, like healthcare providers, are allowed the use the number. 

## Querying Health Insurance Status

You can use Postman to query the health insurance status of a patient:

1. Use the **GET** request `/search/:bsn` in Postman.
2. Replace `:bsn` with the patient's BSN (citizen service number).

You can use one of the following example BSNs for testing:

- 569628132
- 806758911
- 266163192
- 375014033
- 434676756
- 021729993
- 418099614
- 940776365

Enter the BSN in the request URL and click **Send**. The response will show the health insurance status for the specified patient.

{: .important }
> The BSNs used in this workshop are fake, never use your own or someone else's BSN for testing.