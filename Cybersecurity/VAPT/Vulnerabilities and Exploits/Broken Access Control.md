

Broken access control occurs when users can access resources or perform actions beyond their assigned permissions due to flaws in the application's authorization mechanisms. This can lead to unauthorized data exposure, modification, or even system compromise.

---

## **Types of Broken Access Control**

1. **Vertical Privilege Escalation**  
    Occurs when a user gains access to resources or actions meant for higher-privileged users.  
    **Example:** A regular user accessing admin functionalities by modifying session data.
    
2. **Horizontal Privilege Escalation**  
    Happens when a user accesses the resources or data of another user with the same privilege level.  
    **Example:** User A accessing User B's account by changing a user ID in the URL.
    
3. **Insecure Direct Object References (IDOR)**  
    Allows unauthorized access to resources by modifying identifiers directly.  
    **Example:** Accessing `https://example.com/doc/123` instead of `https://example.com/doc/124`.
    
4. **Forced Browsing**  
    Occurs when an attacker accesses unprotected resources by guessing URLs or endpoints.  
    **Example:** Accessing hidden admin pages not linked in the UI.
    

---

## **Mitigation Steps**

1. **Implement Role-Based Access Control (RBAC):**  
    Use well-defined roles and permissions to restrict access to resources.  
    _Example:_ Separate admin and user access levels using a secure authorization framework.
    
2. **Server-Side Enforcement:**  
    Ensure that all access control checks are implemented server-side, not client-side.  
    _Example:_ Validate every request for proper authorization on the backend.
    
3. **Use Least Privilege Principle:**  
    Assign the minimum necessary permissions to users and applications.  
    _Example:_ Grant read-only access where write access is unnecessary.
    
4. **Secure Object References:**  
    Avoid exposing sensitive resource identifiers like database keys; use indirect references (e.g., UUIDs).  
    _Example:_ Replace `/doc/123` with `/doc/secure-id`.
    
5. **Audit and Logging:**  
    Monitor and log access attempts, particularly failed or suspicious ones.  
    _Example:_ Alert when a user attempts to access unauthorized endpoints repeatedly.
    
6. **Test and Validate Access Controls:**  
    Conduct regular penetration testing and code reviews to identify vulnerabilities.  
    _Example:_ Simulate attacks like IDOR during testing.
    
7. **Use Secure Frameworks:**  
    Employ frameworks that include built-in access control mechanisms.  
    _Example:_ Use Spring Security for Java or Django's permission system for Python.