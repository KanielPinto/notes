
## **Rules Of Engagement ( ROE )** #rules-of-engagement

The ROE is a document that is created at the initial stages of a penetration testing engagement. The SANS institute has a great example of this document which you can view online [here](https://sansorg.egnyte.com/dl/bF4I3yCcnt/?).

A standard ROE document possesses the following 3 sections :

| **Section** |                                                                                                                   **Description**                                                                                                                    |
| :---------: | :--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------: |
| Permission  |                    This section of the document gives explicit permission for the engagement to be carried out. This permission is essential to legally protect individuals and organisations for the activities they carry out.                     |
| Test Scope  |                  This section of the document will annotate specific targets to which the engagement should apply. For example, the penetration test may only apply to certain servers or applications but not the entire network.                   |
|    Rules    | The rules section will define exactly the techniques that are permitted during the engagement. For example, the rules may specifically state that techniques such as phishing attacks are prohibited, but MITM (Man-in-the-Middle) attacks are okay. |


## **Penetration Test Stages** #pentest-stages

| **Stage**             | **Description**                                                                                                                                                                                                                                                                                                                                                                  |
| :-------------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Information Gathering | This stage involves collecting as much publically accessible information about a target/organisation as possible, for example, OSINT and research.<br><br>**Note:** This does not involve scanning any systems.                                                                                                                                                                  |
| Enumeration/Scanning  | This stage involves discovering applications and services running on the systems. For example, finding a web server that may be potentially vulnerable.                                                                                                                                                                                                                          |
| Exploitation          | This stage involves leveraging vulnerabilities discovered on a system or application. This stage can involve the use of public exploits or exploiting application logic.                                                                                                                                                                                                         |
| Privilege Escalation  | Once you have successfully exploited a system or application (known as a foothold), this stage is the attempt to expand your access to a system. You can escalate horizontally and vertically, where horizontally is accessing another account of the same permission group (i.e. another user), whereas vertically is that of another permission group (i.e. an administrator). |
| Post-exploitation     | This stage involves a few sub-stages:  <br><br>**1.** What other hosts can be targeted (pivoting)<br><br>**2.** What additional information can we gather from the host now that we are a privileged user<br><br>**3.**  Covering your tracks<br><br>**4.** Reporting                                                                                                            |

## **Penetration Testing Methodologies** #pentest-methodology

Common popular PT methodologies include:

### [The Open Source Security Testing Methodology Manual](https://github.com/mtesauro/owasp-wte/blob/master/temp-projects/wte-docs/contents/usr/share/doc/WTE-Documentation/OSSTMM/OSSTMM.3.pdf) : 

The OSSTMM provides a detailed framework of testing strategies for systems, software, applications, communications and the human aspect of cybersecurity.

| **Advantages**                                                                                                                                             | **Disadvantages**                                                                             |
| ---------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------- |
| Covers various testing strategies in-depth.                                                                                                                | The framework is difficult to understand, very detailed, and tends to use unique definitions. |
| Includes testing strategies for specific targets (I.e. telecommunications and networking)                                                                  | _Intentionally left blank._                                                                   |
| The framework is flexible depending upon the organisation's needs.                                                                                         | _Intentionally left blank._                                                                   |
| The framework is meant to set a standard for systems and applications, meaning that a universal methodology can be used in a penetration testing scenario. | _Intentionally left blank._                                                                   |

### [Open Web Application Security Project](https://owasp.org/) : 

The #OWASP framework is a community-driven and frequently updated framework used solely to test the security of web applications and services.

| **Advantages**                                                                    | **Disadvantages**                                                                              |
| --------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------- |
| Easy to pick up and understand.                                                   | It may not be clear what type of vulnerability a web application has (they can often overlap). |
| Actively maintained and is frequently updated.                                    | OWASP does not make suggestions to any specific software development life cycles.              |
| It covers all stages of an engagement: from testing to reporting and remediation. | The framework doesn't hold any accreditation such as CHECK.                                    |
| Specialises in web applications and services.                                     | _Intentionally left blank._                                                                    |

### [NIST Cybersecurity Framework](https://www.nist.gov/cyberframework) :

The #NIST Cybersecurity Framework is a popular framework used to improve an organization's cybersecurity standards and manage the risk of cyber threats. 

| **Advantages**                                                                                                             | **Disadvantages**                                                                                                  |
| -------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------ |
| The NIST Framework is estimated to be used by 50% of American organisations by 2020.                                       | NIST has many iterations of frameworks, so it may be difficult to decide which one applies to your organisation.   |
| The framework is extremely detailed in setting standards to help organisations mitigate the threat posed by cyber threats. | The NIST framework has weak auditing policies, making it difficult to determine how a breach occurred.             |
| The framework is very frequently updated.                                                                                  | The framework does not consider cloud computing, which is quickly becoming increasingly popular for organisations. |
| NIST provides accreditation for organisations that use this framework.                                                     | _Intentionally left blank.  <br>_                                                                                  |
| The NIST framework is designed to be implemented alongside other frameworks.                                               | _Intentionally left blank._                                                                                        |

## **Testing Scopes** #pentest-scopes

#### Black-Box Testing

This testing process is a high-level process where the tester is not given any information about the inner workings of the application or service.

#### Grey-Box Testing

This testing process is the most popular for things such as penetration testing. It is a combination of both black-box and white-box testing processes. The tester will have some **limited** knowledge of the internal components of the application or piece of software.

#### White-Box Testing

This testing process is a low-level process usually done by a software developer who knows programming and application logic. The tester will be testing the internal components of the application or piece of software and, for example, ensuring that specific functions work correctly and within a reasonable amount of time.


## **Vulnerability, Exploit and Payload**

**Exploit:** 
A piece of code that uses a vulnerability present on the target system.

**Vulnerability:** 
A design, coding, or logic flaw affecting the target system. The exploitation of a vulnerability can result in disclosing confidential information or allowing the attacker to execute code on the target system.

**Payload:** 
An exploit will take advantage of a vulnerability. However, if we want the exploit to have the result we want (gaining access to the target system, read confidential information, etc.), we need to use a payload. Payloads are the code that will run on the target system.