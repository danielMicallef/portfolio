---
layout: ../../layouts/BlogPost.astro
title: "API Authentication Explained: Basic Auth, Bearer Tokens & JWTs"
lead: "A comprehensive guide to the three fundamental API authentication methods"
date: 2025-10-14
draft: true
tags: ["Authentication", "API", "Security", "JWT"]
---

This video provides a clear explanation of the three fundamental methods for API authentication: Basic Auth, Bearer Tokens (with opaque tokens), and JSON Web Tokens (JWTs), covering how they work, when to use them, and crucial security considerations.

### The Foundation of API Authentication

* **The Problem** The need for authentication ("Who are you?") arises because **HTTP is stateless** [[00:40](http://www.youtube.com/watch?v=I747kI_y9eQ&t=40)]. Every request is treated as a new, clean slate, requiring the client to prove its identity each time [[00:50](http://www.youtube.com/watch?v=I747kI_y9eQ&t=50)].
* The video introduces the three methods: Basic Auth (simplest), Bearer Tokens (most common transport), and JWTs (self-contained) [[01:17](http://www.youtube.com/watch?v=I747kI_y9eQ&t=77)].

### 1. Basic Authentication

* **How it works:** The client joins the username and password with a colon (`:`), encodes the result in **Base64**, and sends it in the `Authorization` header with every request [[01:33](http://www.youtube.com/watch?v=I747kI_y9eQ&t=93)].
* **Security Risk:** **Base64 is encoding, not encryption** [[01:49](http://www.youtube.com/watch?v=I747kI_y9eQ&t=109)]. If sent over plain HTTP, the password is in unencrypted form.
* **Best Practice:** **Must only be used over HTTPS** (TLS encryption protects the credentials) [[02:30](http://www.youtube.com/watch?v=I747kI_y9eQ&t=150)].
* **Use Cases:** Internal tools, local development, or simple machine-to-machine communication where the network is controlled [[02:53](http://www.youtube.com/watch?v=I747kI_y9eQ&t=173)].

### 2. Bearer Tokens (Opaque Tokens)

* **The Concept:** The term **Bearer** is the transport mechanism ("Give this to whoever holds it"), and the token is the content [[03:10](http://www.youtube.com/watch?v=I747kI_y9eQ&t=190)]. An **opaque token** is a random string that contains no information itself [[04:16](http://www.youtube.com/watch?v=I747kI_y9eQ&t=256)].
* **How it works:**
    1.  Client sends credentials once to the server.
    2.  Server validates, generates a random token, and stores it in a database.
    3.  Client sends the token in the `Authorization: Bearer <token>` header on subsequent requests [[03:45](http://www.youtube.com/watch?v=I747kI_y9eQ&t=225)].
    4.  The server must query the database on every single request to validate the token and identify the user [[04:08](http://www.youtube.com/watch?v=I747kI_y9eQ&t=248)].
* **Advantages:** You're not sending the password repeatedly, tokens can be instantly revoked, and they can have expiration times [[04:31](http://www.youtube.com/watch?v=I747kI_y9eQ&t=271)].
* **Trade-off:** The database lookup on every request can be a **performance bottleneck** in high-traffic applications, and it requires shared session storage (e.g., Redis) for horizontally scaled API servers [[04:46](http://www.youtube.com/watch?v=I747kI_y9eQ&t=286)].

### 3. JSON Web Tokens (JWTs)

* **The Structure:** A JWT is a self-contained token with three parts separated by dots: **Header**, **Payload**, and **Signature** [[05:20](http://www.youtube.com/watch?v=I747kI_y9eQ&t=320)].
* **Payload (Data):** Contains "claims" (information about the user, like ID, roles, and expiration time) [[05:45](http://www.youtube.com/watch?v=I747kI_y9eQ&t=345)].
    * **Critical Security Note:** The payload is only **Base64 encoded, not encrypted**. Anyone can decode and read it (e.g., using jwt.io). You must **never** put sensitive data like passwords or credit card numbers inside a JWT [[06:10](http://www.youtube.com/watch?v=I747kI_y9eQ&t=370)].
* **Signature (Security):** The server cryptographically signs the header and payload with a secret key [[06:43](http://www.youtube.com/watch?v=I747kI_y9eQ&t=403)]. This hash makes the JWT **tamper-proof**; if anyone changes the payload, the signature won't match, and the server will reject it [[06:51](http://www.youtube.com/watch?v=I747kI_y9eQ&t=411)].
* **Game-Changing Advantage:** The server doesn't need to look up the database on every request. It only verifies the signature mathematically, which is typically 5 to 10 times faster. This makes JWTs **stateless** and perfect for horizontal scaling [[07:14](http://www.youtube.com/watch?v=I747kI_y9eQ&t=434)].
* **Trade-off (Revocation):** Because they are stateless, revoking a JWT before its expiration is difficult and requires additional infrastructure (like a token blacklist), which can negate the stateless advantage [[07:40](http://www.youtube.com/watch?v=I747kI_y9eQ&t=460)].
    * **The Solution:** Use **short-lived access tokens** (e.g., 15 minutes) paired with longer-lived **refresh tokens** that are stored in the database and can be revoked [[08:12](http://www.youtube.com/watch?v=I747kI_y9eQ&t=492)].

### Key Security Mistakes to Avoid

The video highlights five common security mistakes, regardless of the method used:

1.  **Always use HTTPS** [[09:39](http://www.youtube.com/watch?v=I747kI_y9eQ&t=579)].
2.  **Token Storage:** Avoid local storage (vulnerable to XSS). Use **HTTP-only cookies** with the `SameSite` attribute set to `strict` or `lax` to defend against both XSS and CSRF [[09:58](http://www.youtube.com/watch?v=I747kI_y9eQ&t=598)].
3.  **Expiration:** Use short-lived access tokens [[10:34](http://www.youtube.com/watch?v=I747kI_y9eQ&t=634)].
4.  **Cryptography:** Never roll your own crypto; use established, audited libraries [[10:42](http://www.youtube.com/watch?v=I747kI_y9eQ&t=642)].
5.  **JWT Algorithm Verification:** Always explicitly specify and **whitelist the expected algorithms** when verifying a JWT to prevent "algorithm confusion" attacks [[11:00](http://www.youtube.com/watch?v=I747kI_y9eQ&t=660)].

### Practical Decision Framework

* **Basic Auth + HTTPS:** Use for internal tools with controlled access [[11:40](http://www.youtube.com/watch?v=I747kI_y9eQ&t=700)].
* **JWTs:** Use if you need to scale horizontally with multiple servers (stateless, fast) [[11:56](http://www.youtube.com/watch?v=I747kI_y9eQ&t=716)].
* **Opaque Bearer Tokens:** Use for simpler applications where the database lookup is not a performance problem, as they are easier to implement and revoke [[12:03](http://www.youtube.com/watch?v=I747kI_y9eQ&t=723)].

***
**Video Link:** [API Authentication Explained (Finally) — Basic Auth, Bearer & JWT](http://www.youtube.com/watch?v=I747kI_y9eQ)
