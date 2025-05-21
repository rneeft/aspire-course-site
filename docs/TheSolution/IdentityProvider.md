---
title: Identity Provider
layout: home
parent: The Solution
nav_order: 1.2
---

# The Identity Provider

An **Identity Provider** (IdP) is a service responsible for managing user identities and handling authentication. Its main responsibility is to verify user credentials and issue tokens that other services can trust. In this workshop, the Identity Provider authenticates users by email and password, and issues a **JSON Web Token (JWT)** upon successful login.

## Creating a User

To create a user, use the `/identity/create` request in Postman:

1. Open the `/identity/create` request in the Postman collection.
2. In the **Body** section, fill in the `username`, `email`, and `password`.
3. Click **Send**.

{: .highlight }
When using `workshop.nl` domain the user becomes an admin which are authorisated to create Health insurers.

**Password Security:**  
Passwords are not stored in plain text. Instead, they are saved as a salted hash using the **argon2** algorithm. This is important because hashing with a salt protects user passwords even if the database is compromised, making it much harder for attackers to recover the original passwords.

## Logging In and Retrieving a JWT

After creating a user, you can log in and retrieve a JWT:

1. Use the `/identity/login` request in Postman.
2. In the **Body** section, enter the `email` and `password`.
3. Click **Send**. If the credentials are correct, a JWT will be returned.

**Example JWT:**
```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJodHRwOi8vc2NoZW1hcy54bWxzb2FwLm9yZy93cy8yMDA1LzA1L2lkZW50aXR5L2NsYWltcy9lbWFpbGFkZHJlc3MiOiJlbWFpbC5lbWFpbEBleGFtcGxlLmNvbSIsImh0dHA6Ly9zY2hlbWFzLm1pY3Jvc29mdC5jb20vd3MvMjAwOC8wNi9pZGVudGl0eS9jbGFpbXMvcm9sZSI6InVzZXIiLCJodHRwOi8vc2NoZW1hcy54bWxzb2FwLm9yZy93cy8yMDA1LzA1L2lkZW50aXR5L2NsYWltcy9uYW1laWRlbnRpZmllciI6IjIiLCJleHAiOjE3NDc4NDY5NDEsImlzcyI6ImxvY2FsaG9zdCIsImF1ZCI6IkFwaSJ9.i8gckI8JCbohip-I1ouG2DV7gHz3hBS6iGRwUKjgoEc
```

You can inspect the contents of a JWT at [JWT.IO](https://jwt.io/). The 256-bit secret used to sign the JWT can be found in the `appsettings.json` file.

## Next section
Now that a user is created, lets create a [Health insurer](HealthInsurers).