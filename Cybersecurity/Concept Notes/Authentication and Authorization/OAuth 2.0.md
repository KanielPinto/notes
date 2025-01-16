
### **What is OAuth 2.0?

OAuth 2.0 is an open standard framework designed to enable secure delegated access. It allows applications to access resources on behalf of a user without sharing their credentials. For example, a user can allow a third-party app to post to their social media account without sharing their password.

### **OAuth 2.0 Authorization Flow and Roles

#### Roles:

1. **Resource Owner**: The entity capable of granting access to a protected resource (e.g., a user who owns a Google Drive file).
2. **Client**: The application requesting access to resources on behalf of the resource owner (e.g., a third-party app like Canva requesting access to Google Drive).
3. **Authorization Server**: Issues access tokens after authenticating the resource owner and obtaining authorization (e.g., Google's OAuth server).
4. **Resource Server**: Hosts the protected resources and verifies access tokens to fulfill requests (e.g., the Google Drive API).

#### **Authorization Flow:

1. The client (e.g., a calendar app) requests authorization from the resource owner.
2. The resource owner grants or denies the request (e.g., the user approves access through a Google login page).
3. If granted, the client receives an authorization grant (e.g., a temporary code).
4. The client exchanges the grant with the authorization server for an access token.
5. The client uses the access token to access the protected resources on the resource server (e.g., retrieving calendar data from Google Calendar).

### **Registration

Before using OAuth 2.0, the client must register with the authorization server. During registration, the client receives credentials, such as a **client ID** and **client secret**, and specifies redirect URIs. For example, a developer building a weather app might register their app with Facebook to enable Facebook Login.

### **Authorization Server and Resource Server

- **Authorization Server**: Handles user authentication and issues access tokens upon successful authorization (e.g., GitHub's authorization server).
- **Resource Server**: Validates access tokens and serves protected resources to authorized clients (e.g., GitHub API providing repository data).

### **Access Token

An access token is a credential issued to the client by the authorization server. It:

- Represents the resource owner’s authorization.
- Is sent in API requests to access protected resources.
- Can have a limited lifespan and specific scopes. For instance, a token might allow access to read a user's emails but not to send new ones.

### **Scope and Permitted Resource Calls

- **Scope**: Defines the permissions or access levels granted to the client. For example:
    - `read:user` (access to user profile data)
    - `write:email` (permission to send emails)
- Clients must request scopes explicitly during authorization. For instance, an email app might request `read:email` and `send:email` scopes to perform its functions.

### **OAuth is Not an Authentication Protocol

OAuth 2.0 is a framework for **authorization**, not **authentication**. For example, while OAuth can let a travel app access a user's calendar, it doesn’t verify the user's identity for login. To enable authentication, protocols like OpenID Connect build on OAuth 2.0.

### **Social Login

A popular use case of OAuth 2.0 is enabling users to log in to an application using credentials from another platform (e.g., "Log in with Google"). This eliminates the need to create new accounts and passwords for every service and allows users to link multiple apps to a single account.

### **4 Grant Types

#### 1. Authorization Code Grant

**Use Case**: Server-side applications.

**How It Works**:

1. The client directs the user to the authorization server.
2. The user logs in and grants permission.
3. The server redirects the user back to the client with an authorization code.
4. The client exchanges the code for an access token.

**Example**:

- A project management app like Asana allows users to integrate their Google Calendar.
- The app redirects the user to Google, where they log in and approve the connection.
- Google returns an authorization code to Asana, which exchanges it for an access token to retrieve calendar events.

**Bottom Line**: Secure as tokens are exchanged server-side; avoids exposing credentials or tokens to the browser.

---

#### 2. Implicit Grant

**Use Case**: Single-page or client-side applications.

**How It Works**:

1. The client requests authorization directly from the resource owner.
2. Upon approval, the client receives an access token directly in the browser.

**Example**:

- A weather widget on a website might request access to a user’s location data via a third-party service.
- The token is returned directly to the browser to enable access to location APIs.

**Bottom Line**: Simpler but less secure; access tokens are exposed to the browser and vulnerable to interception.

---

#### 3. Resource Owner Password Credentials Grant

**Use Case**: Trusted clients like first-party applications.

**How It Works**:

1. The resource owner provides credentials directly to the client.
2. The client exchanges the credentials with the authorization server for an access token.

**Example**:

- A mobile banking app owned by the same bank as the resource server may use this method to directly authenticate users.

**Bottom Line**: Requires high trust; not recommended for third-party applications.

---

#### 4. Client Credentials Grant

**Use Case**: Machine-to-machine communication.

**How It Works**:

1. The client authenticates with the authorization server using its client ID and secret.
2. An access token is issued without involving a resource owner.

**Example**:

- A backend service querying another service (e.g., a payment gateway querying a fraud detection API).

**Bottom Line**: Secure and suitable for backend services without user involvement.

### **Complexity of OAuth 2.0

OAuth 2.0 is a flexible framework, but its implementation can be complex due to:

- Token management (e.g., refreshing expired tokens).
- Ensuring secure storage and transmission of tokens (e.g., using HTTPS).
- Handling refresh tokens and token revocation.
- Implementing proper scope restrictions.

### **OAuth 2.0 Is a Framework

OAuth 2.0 provides a flexible foundation but does not define:

- Authentication flows (e.g., validating a user’s identity).
- Token formats (e.g., JWT, opaque tokens).
- Standardized error messages.
- Methods for token revocation.

### **Disadvantages

1. **Complexity**: Requires careful implementation to ensure security.
2. **Security Risks**: Vulnerable to attacks like token interception and replay if improperly implemented.
3. **No Authentication Guarantee**: Lacks built-in mechanisms for user authentication.

### **Challenges of OAuth 2.0 Implementation

1. **Token Security**: Ensuring tokens are securely stored and transmitted (e.g., using secure storage mechanisms on mobile devices).
2. **Implementation Variability**: Differences in server configurations can lead to inconsistent behaviors (e.g., varying interpretations of scope).
3. **Phishing Risks**: Users might be tricked into granting permissions to malicious applications (e.g., via fake authorization screens).
4. **Complex Revocation**: Ensuring proper token revocation and session management, especially in scenarios involving multiple devices.
5. **Evolving Standards**: Keeping up with best practices and updates (e.g., PKCE, JWT).

### **[[OAuth 2.0 Threats]]**

### **In Summary

OAuth 2.0 provides a secure and scalable mechanism for authorization, enabling applications to access resources on behalf of users without sharing credentials. For example, it powers integrations like linking a fitness app with a health tracking service. While flexible, its correct implementation is critical to avoid security pitfalls.

---

