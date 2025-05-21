---
title: Health Insurers
layout: home
parent: The Solution
nav_order: 1.3
---

# Health Insurer
In our system health insurance data is processed and can be queried by healthcare providers like docters, hospitals, dentists etc. Health insurers can upload their health care policies. Lets create a health insurer in the system. 

To create a health insurer:

1. In Postman, use the **POST** request to `/HealthInsurers`.
2. In the **Auth** tab, select **Bearer Token** and paste your JWT.
3. In the **Body**, specify the `name` of the health insurer.
4. Click **Send**.

Create two health insurers.

After creating a health insurer, you can retrieve it using the **GET** request to `/HealthInsurers`. The response will include an `apiKey`, which is the second way to authenticate in the system.

## X-API-KEY Authentication

With X-API-KEY authentication, you include the `apiKey` in the request header:

```
X-API-KEY: your-api-key-here
```

This allows clients to authenticate without a JWT, using the API key provided for the health insurer. This is useful for service-to-service communication or for clients that cannot use JWTs.
