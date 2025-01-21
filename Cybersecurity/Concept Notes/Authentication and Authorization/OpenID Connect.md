
### **OpenID Connect vs OAuth 2.0

- **OAuth 2.0**: A protocol designed for authorization. It allows third-party applications to obtain limited access to a user's resources without exposing their credentials.
- **OpenID Connect (OIDC)**: An identity layer built on top of OAuth 2.0 that enables authentication (verifying the user's identity) in addition to authorization.

#### Key Differences
| Feature              | OAuth 2.0                | OpenID Connect                 |
| -------------------- | ------------------------ | ------------------------------ |
| Purpose              | Authorization            | Authentication + Authorization |
| Token Types          | Access Token             | Access Token, ID Token         |
| User Authentication  | Not inherently supported | Supported                      |
| Identity Information | Not provided             | Provided via ID Token          |
| Protocol Extensions  | Generic                  | Identity-focused extensions    |

---

### **Downsides of Standalone OAuth 2.0

1. **No Built-in Authentication**:
   - OAuth 2.0 does not verify a user's identity, leading to potential misuse when misapplied for authentication purposes.

2. **Vulnerable Implementations**:
   - Without standardized guidance (like OIDC), developers might implement insecure authentication flows.

3. **Lack of Identity Data**:
   - OAuth 2.0 only provides authorization data, requiring custom extensions for identity.

4. **Interoperability Issues**:
   - Standalone OAuth 2.0 implementations lack a uniform standard for handling identity, causing inconsistencies.

---

### **OpenID Connect Identity

- **Identity Provider (IdP)**: The entity that authenticates the user and issues tokens.
- **Relying Party (RP)**: The application or service relying on the IdP for identity verification.
- **ID Token**: A JSON Web Token ([[JWT]]) containing user identity claims, signed by the IdP.

---

### **OpenID Connect Detailed Example

1. **User Login**:
   - The user accesses the Relying Party (RP) and initiates login.

2. **Authorization Request**:
   - The RP redirects the user to the Identity Provider (IdP) with an OAuth 2.0 authorization request.

3. **User Authentication**:
   - The user authenticates with the IdP (e.g., using username/password or multi-factor authentication).

4. **Token Issuance**:
   - The IdP issues an ID Token and Access Token, redirecting the user back to the RP.

5. **Token Validation**:
   - The RP validates the ID Token's signature and retrieves user information.

6. **Session Establishment**:
   - The RP establishes a session for the authenticated user.

---

### **Standard Claims and Profile Parameter

#### Standard Claims
- **Sub**: Unique identifier for the user.
- **Name**: Full name of the user.
- **Preferred_username**: User's preferred username.
- **Email**: User's email address.
- **Picture**: URL to the user's profile picture.

#### Profile Parameter
- The `profile` scope requests additional standard claims like `birthdate`, `locale`, and `gender`.

---

### **Additional Parameters

- **Nonce**: A unique string to prevent replay attacks.
- **Acr**: Authentication Context Class Reference, indicating authentication strength.
- **Amr**: Authentication Methods Reference, listing methods used (e.g., password, OTP).

---
### **OIDC Code Flow

#### Step 1: Authorization Request

1. The client (Relying Party) redirects the end user to the Authorization Server.
2. The request includes the following parameters:
   - `response_type=code`: Indicates the Authorization Code Flow is being used.
   - `scope=openid`: Requests OpenID Connect functionality.
   - `client_id`: Identifies the client application.
   - `redirect_uri`: URL where the Authorization Server will send the user after authorization.
   - `state`: A unique string to prevent CSRF attacks.
   - `nonce`: A unique value to prevent replay attacks and associate the ID Token with the current session.

#### Example Request

```http
GET /authorize
  ?response_type=code
  &client_id=client123
  &redirect_uri=https://example.com/callback
  &scope=openid profile email
  &state=abc123
  &nonce=xyz456
HTTP/1.1
Host: auth.example.com
```

---

#### Step 2: User Authentication and Consent

1. The Authorization Server authenticates the user using credentials (e.g., username/password) or multi-factor authentication (MFA).
2. The user grants consent to share identity information and grant the requested permissions.

---

#### Step 3: Authorization Code Issuance

1. After successful authentication and consent, the Authorization Server redirects the user back to the client’s `redirect_uri`.
2. The redirection includes the `authorization_code` and `state` (to match the client’s request).

#### Example Redirect

```
HTTP/1.1 302 Found
Location: https://example.com/callback
  ?code=auth_code_123
  &state=abc123
```

---

#### Step 4: Token Exchange

1. The client sends a POST request to the Authorization Server’s token endpoint to exchange the authorization code for tokens.
2. The request includes:
   - `grant_type=authorization_code`: Specifies the flow type.
   - `code`: The authorization code received in the previous step.
   - `redirect_uri`: The same value used in the authorization request.
   - `client_id` and `client_secret`: Used to authenticate the client.

#### Example Token Request

```http
POST /token HTTP/1.1
Host: auth.example.com
Content-Type: application/x-www-form-urlencoded

grant_type=authorization_code
&code=auth_code_123
&redirect_uri=https://example.com/callback
&client_id=client123
&client_secret=secret456
```

### Token Response

The Authorization Server responds with:
- **ID Token**: A JSON Web Token ([[JWT]]) containing user identity claims (e.g., `sub`, `email`, `name`).
- **Access Token**: Used to access protected resources (e.g., `userinfo` endpoint).
- **Refresh Token**: Optionally issued to obtain new tokens without user interaction.

#### Example Token Response

```json
{
  "id_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "access_token": "SlAV32hkKG...",
  "refresh_token": "8xLOxBtZp8...",
  "token_type": "Bearer",
  "expires_in": 3600
}
```

---

#### Step 5: ID Token Validation

1. The client validates the `ID Token` by:
   - Verifying the token’s signature using the Authorization Server’s public keys.
   - Checking the `aud` (audience) claim to ensure it matches the client’s `client_id`.
   - Validating the `nonce` to prevent replay attacks.

2. Once validated, the client extracts user identity claims (e.g., `sub`, `name`, `email`) from the `ID Token`.

---

#### Step 6: Accessing User Information

1. The client uses the `Access Token` to call the `userinfo` endpoint of the Authorization Server.
2. The response contains additional user claims (if authorized), such as `address`, `phone_number`, etc.

#### Example userinfo Request

```http
GET /userinfo HTTP/1.1
Host: auth.example.com
Authorization: Bearer SlAV32hkKG...
```

#### Example userinfo Response

```json
{
  "sub": "1234567890",
  "name": "John Doe",
  "email": "john.doe@example.com",
  "picture": "https://example.com/johndoe.jpg"
}
```

#### Comparison to OAuth 2.0

- **ID Token**: Unique to OIDC, providing user identity claims in a signed JWT.
- **Nonce**: Adds security and session binding to prevent replay attacks.
- **Scope**: OIDC introduces the `openid` scope to indicate authentication.
- **Claims**: While OAuth 2.0 focuses on authorization, OIDC enriches responses with user identity claims.

---

### **Userinfo Endpoint

- **Purpose**: Provides additional user claims.
- **Access Method**: Requires a valid Access Token.
- **Response**: JSON object with claims such as `name`, `email`, and `picture`.

---

### **ID Token vs Access Token in OpenID Connect

| Feature  | ID Token              | Access Token                  |
| -------- | --------------------- | ----------------------------- |
| Purpose  | Identity information  | Resource access authorization |
| Format   | [[JWT]]               | Can be opaque or JWT          |
| Audience | Relying Party (RP)    | Resource Server               |
| Lifetime | Typically short-lived | Configurable                  |

---

### **OpenID Connect Protocol Suite and Its 3 Levels

1. **Basic**: Core functionality for authentication.
2. **Implicit**: Token issuance via browser redirects.
3. **Hybrid**: Combines Authorization Code and Implicit flows.

---

### **Summary

- **OAuth 2.0** is for authorization; **OIDC** adds authentication.
- OIDC introduces ID Tokens, user claims, and authentication flows.
- The protocol standardizes identity handling, enhancing security and interoperability.

