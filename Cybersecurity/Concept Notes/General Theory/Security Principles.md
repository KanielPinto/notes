
*No solution is 100% secure. All we can do is try to improve our security posture to make it more difficult for our adversaries to gain access.*  

### **Security Triad ( CIA )**  

- ***Confidentiality*** ensures that only the intended persons or recipients can access the data.
- ***Integrity*** aims to ensure that the data cannot be altered; moreover, that we can detect any alteration if it occurs.
- ***Availability*** aims to ensure that the system or service is available when needed.

The emphasis does not need to be the same on all three security functions. One example would be a university announcement; although it is usually not confidential, the document’s integrity is critical.

If we go a step beyond the security triad we have two more elements :
- ***Authenticity*** is about ensuring that the document/file/data is from the claimed source.
- ***Nonrepudiation*** ensures that the original source cannot deny that they are the source of a particular document/file/data.

#### **Parkerian Hexad** 

In 1998, Donn Parker proposed the Parkerian Hexad, a set of six security elements. They are:

1. Availability
2. Utility
3. Integrity
4. Authenticity
5. Confidentiality
6. Possession

- ***Utility*** focuses on the usefulness of the information. For instance, a user might have lost the decryption key to access a laptop with encrypted storage. Meaning, although still available, the information is in a form that is not useful, i.e., of no utility.

- ***Possession*** requires that we protect the information from unauthorized taking, copying, or controlling.

#### **Anti-Security Triad ( DAD )**

- ***Disclosure*** is the opposite of confidentiality.
- ***Alteration*** is the opposite of Integrity. 
- ***Destruction/Denial*** is the opposite of Availability.

#### **Balance**

Protecting confidentiality and integrity to an extreme can restrict availability, and increasing availability to an extreme can result in losing confidentiality and integrity. Good security principles implementation requires a balance between the three.


### **Foundational Security Models**

#### **Bell-LaPadula Model** :
- Aims for *Confidentiality*
- **Simple Security Property**: This property is referred to as “no read up”
- **Star Security Property**: This property is referred to as “no write down”
- **“Write up, Read down”**

#### **Biba Model** : 
- Aims for *Integrity*
- **Simple Integrity Property**: This property is referred to as “no read down”
- **Star Integrity Property**: This property is referred to as “no write up”
- **“Read up, Write down”**

#### **Clark-Wilson Model** :
- Aims for *Integrity*
- **Constrained Data Item (CDI)**: This refers to the data type whose integrity we want to preserve.
- **Unconstrained Data Item (UDI)**: This refers to all data types beyond CDI, such as user and system input.
- **Transformation Procedures (TPs)**: These procedures are programmed operations, such as read and write, and should maintain the integrity of CDIs.
- **Integrity Verification Procedures (IVPs)**: These procedures check and ensure the validity of CDIs.



### **Defense-In-Depth** 

It refers to creating a security system of multiple levels; hence it is also called ***Multi-Level Security***.

### **ISO/IEC 19249**

#### **ISO/IEC 19249 lists five _architectural_ principles :**

1. ***Domain Separation*** : Every set of related components is grouped as a single entity; components can be applications, data, or other resources. Each entity will have its own domain and be assigned a common set of security attributes.

2. ***Layering*** : When a system is structured into many abstract levels or layers, it becomes possible to impose security policies at different levels; moreover, it would be feasible to validate the operation. Let’s consider the OSI (Open Systems Interconnection) model wherein each layer provides specific services to the layer above it. This layering makes it possible to impose security policies and easily validate that the system is working as intended. Layering relates to Defense in Depth.

3. ***Encapsulation*** : In object-oriented programming (OOP), we hide low-level implementations and prevent direct manipulation of the data in an object by providing specific methods for that purpose. For example, if you have a clock object, you would provide a method `increment()` instead of giving the user direct access to the `seconds` variable. The aim is to prevent invalid values for your variables. Similarly, in larger systems, you would use (or even design) a proper Application Programming Interface (API) that your application would use to access the database.

4. ***Redundancy*** : This principle ensures availability and integrity. There are many examples related to redundancy. Consider the case of a hardware server with two built-in power supplies: if one power supply fails, the system continues to function. 

5. ***Virtualization*** : The concept of virtualization is sharing a single set of hardware among multiple operating systems. Virtualization provides sandboxing capabilities that improve security boundaries, secure detonation, and observance of malicious programs.

#### **ISO/IEC 19249 teaches five _design_ principles :**

1. ***Least Privilege*** : The principle of least privilege teaches that you should provide the least amount of permissions for someone to carry out their task and nothing more. For example, if a user needs to be able to view a document, you should give them read rights without write rights.

2. ***Attack Surface Minimization*** : Every system has vulnerabilities that an attacker might use to compromise a system. Some vulnerabilities are known, while others are yet to be discovered. These vulnerabilities represent risks that we should aim to minimize. For example, in one of the steps to harden a Linux system, we would disable any service we don’t need.

3. ***Centralized Parameter Validation*** : Many threats are due to the system receiving input, especially from users. Invalid inputs can be used to exploit vulnerabilities in the system, such as denial of service and remote code execution. Therefore, parameter validation is a necessary step to ensure the correct system state. Considering the number of parameters a system handles, the validation of the parameters should be centralized within one library or system.

4. _**Centralized General Security Services**_: As a security principle, we should aim to centralize all security services. For example, we would create a centralized server for authentication. Of course, you might take proper measures to ensure availability and prevent creating a single point of failure.

5. ***Preparing for Error and Exception Handling*** : This principle teaches that the systems should be designed to fail safe; for example, if a firewall crashes, it should block all traffic instead of allowing all traffic. Moreover, we should be careful that error messages don’t leak information that we consider confidential, such as dumping memory content that contains information related to other customers.

### **Trust**

***Trust but Verify*** : This principle teaches that we should always verify even when we trust an entity and its behavior. An entity might be a user or a system. Verifying usually requires setting up proper logging mechanisms; verifying indicates going through the logs to ensure everything is normal. In reality, it is not feasible to verify everything.

***Zero Trust*** : This principle treats trust as a vulnerability, and consequently, it caters to insider-related threats. Every entity is considered adversarial until proven otherwise. Zero trust does not grant trust to a device based on its location or ownership. Authentication and authorization are required before accessing any resource. As a result, if any breach occurs, the damage would be more contained if a zero trust architecture had been implemented.

There is a limit to how much we can apply zero trust without negatively impacting a business; however, this does not mean that we should not apply it as long as it is feasible.

### **Vulnerability vs. Threat vs. Risk**

- ***Vulnerability*** : Vulnerable means susceptible to attack or damage. In information security, a vulnerability is a weakness.
- ***Threat*** : A threat is a potential danger associated with this weakness or vulnerability.
- ***Risk*** : The risk is concerned with the likelihood of a threat actor exploiting a vulnerability and the consequent impact on the business.

A showroom with doors and windows made of standard glass suffers a weakness, or _vulnerability_, due to the nature of glass. Consequently, there is a _threat_ that the glass doors and windows can be broken. The showroom owners should contemplate the _risk_, i.e. the likelihood that a glass door or window gets broken and the resulting impact on the business.

### **Shared Responsibility**

Various aspects are required to ensure proper security. They include hardware, network infrastructure, operating systems, applications, etc. However, customers using cloud services have different access levels depending on the cloud services they use. For example, an Infrastructure as a Service (IaaS) user has complete control (and responsibility) over the operating system.

On the other hand, a Software as a Service (SaaS) user has no direct access to the underlying operating system. Consequently, achieving security in a cloud environment necessitates both the cloud service provider and the user to do their parts. The Shared Responsibility Model is a cloud security framework to ensure that each party is aware of its responsibility.

### **References**

- https://tryhackme.com/r/room/securityprinciples
