
APIs provide the interface between the backend and the frontend. 

Designed to be more transparent, this also increases the potential of vulnerabilities.

Major consideration on how security is implemented is distinction between private and public.
private targets internal developers, public external developers, partner is a hybrid of private and public means it targets certain developers working in partnership.

### **API Security Domain**

In a typical context, the functioning of an API depends on the following :

#### 3 Components :
- The API itself.
- A portal for interacting with and configuring the API.
- An application utilizing the API.

#### 3 Users : 
- The End User who accesses an application which utilizes the API.
- The Developer who creates the app and interacts with a portal for the purpose of configuring and integrating the API within the app.
- The API Administrator who creates the API as well as the portal


![[Standard-API-Interaction.png]]


Vulnerabilities can arise at all the points of interaction between the uses and components but can be easy to overlook. This coupled with the fact that APIs shift the granularity boundary from secure internal tiers to the client application results in an increase in attack surface.


### **API Security Risks**

The top security risks to APIs are cataloged within the **[OWASP API Security Top 10](https://owasp.org/API-Security/editions/2023/en/0x11-t10/)**.

**As a whole the most standard measures for ensuring the security of APIs include : **
- Securing documentation if you don't intend your API to be publicly accessible.
- Ensuring  documentation is kept up to date so that legitimate testers have full visibility of the API's attack surface.
- Applying an allowlist of permitted HTTP methods.
- Validating that the content type is expected for each request or response.
- Using generic error messages to avoid giving away information that may be useful for an attacker.
- Using protective measures on all versions of your API, not just the current production version.

**These can be encapsulated under the following security functions**:
- Rate Limiting
- Message Validation
- Encryption and Signing
- Proper Access Control

Authentication and Authorization for APIs are mainly implemented utilizing the **[[OAuth 2.0]]** and **[[OpenID Connect]]** standards.