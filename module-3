# Module 3: Security, Access, and Credential Control

**Instructor:** Ishmael Abayeteye, Data Engineer at AfriqueTek
**Course Level:** Executive / Professional Certification
**Duration:** 3 - 4 Hours
**Prerequisites:** Completion of Module 2.

---

## 1.0 Introduction: The "Fortress" Mintality

### 1.1 Welcome
Welcome to Module 3. We have built the Strategy (Module 1) and ensured the Data is High Quality (Module 2). Now, we must lock the doors.
In the Ghanaian banking sector, we say: *"Trust is hard to gain, but easy to lose."* A single data breach can destroy a 50-year-old reputation overnight.

### 1.2 The Local Threat Landscape
Why is this module critical for *us*?
*   **Global Threat:** Russian/Chinese state-sponsored hackers attacking infrastructure.
*   **Local Threat:** "Sakawa" (Cyber fraud) evolution. It has moved from simple Yahoo scams to sophisticated **SIM Swaps**, **Mobile Money (MoMo) Interface attacks**, and **Social Engineering** of bank staff.
*   **The Difference:** In Europe, they worry about complex malware. In West Africa, we worry about an underpaid employee selling their password for GHS 5,000.

**Module Learning Outcomes:**
1.  **Defend:** Apply the **CIA Triad** to real-world scenarios.
2.  **Control:** Design an **RBAC (Role-Based Access Control)** matrix for a FinTech app.
3.  **Authenticate:** Evaluate the risks of SMS-based 2FA in a region prone to SIM Swapping.

---

## 2.0 Data Security: The CIA Triad

### 2.1 The Holy Trinity of InfoSec
Every security policy you write must satisfy one of these three: **Confidentiality**, **Integrity**, or **Availability**.

#### 1. Confidentiality
*   **Definition:** Only authorized people can see the data.
*   **Global Standard:** Encryption (AES-256 for data at rest, TLS 1.2+ for data in transit).
*   **Local Context:** **Data Masking**.
    *   *Scenario:* A Customer Service Agent in a Telco call center needs to verify a caller.
    *   *Policy:* Do they need to see the *full* ID number? No.
    *   *Implementation:* Mask it: `GHA-720-****`. This prevents the agent from writing down the ID to commit fraud later.

#### 2. Integrity
*   **Definition:** The data has not been altered or tampered with.
*   **Global Standard:** Hashing (SHA-256).
*   **Local Threat:** **The "Ledger Injector."** A database admin directly editing the SQL table to increase a MoMo wallet balance.
*   **Control:** **Immutable Audit Logs**. Logs that cannot be deleted, even by the Admin.

#### 3. Availability
*   **Definition:** The data is accessible when needed.
*   **Local Infrastructure Challenge:** **"Dumsor" (Power Instability).**
    *   *Global View:* Availability means "Protection against DDoS attacks."
    *   *Local View:* Availability means "Do we have diesel in the backup generator?"
    *   *Governance Rule:* Your Disaster Recovery (DR) plan must include fuel logistics, not just code backups.

**Table 3.1: The CIA Triad in Action**

| Principle | The Attack (What goes wrong) | The Defense (Governance Control) |
| :--- | :--- | :--- |
| **Confidentiality** | A leaked Excel sheet of high-net-worth clients. | **DLP (Data Loss Prevention)** tools that block emailing sensitive files. |
| **Integrity** | Changing a loan status from "Default" to "Paid." | **Hashing & Write-Once Logs.** |
| **Availability** | The Server Room overheats because AC failed. | **Redundancy** (Secondary Site in the Cloud). |

---

## 3.0 Access Control Models: Who Keys the Gate?

### 3.1 Authorization vs. Authentication
*   **Authentication (AuthN):** Who are you? (Login).
*   **Authorization (AuthZ):** What are you allowed to do? (Permissions).

### 3.2 RBAC: Role-Based Access Control
This is the standard for 90% of organizations. You give permissions to a "Role," not a person.
*   **Scenario:** A bank branch in Takoradi.
*   **Roles:** `Teller`, `Branch Manager`, `Auditor`.

**Table 3.2: RBAC Matrix Example**

| Action | Role: Teller | Role: Manager | Role: Auditor |
| :--- | :--- | :--- | :--- |
| **View Balance** | YES | YES | YES |
| **Deposit Cash** | YES | YES | NO |
| **Approve Loan (>10k)** | NO | YES | NO |
| **Delete Account** | NO | NO | NO (Only HQ) |

### 3.3 ABAC: Attribute-Based Access Control
RBAC is often too static. **ABAC** is dynamic. It looks at *Context*.
*   **The Rule:** "Allow Access IF (Role = Teller) AND (Location = Branch Office) AND (Time = 8am-5pm)."
*   **Why we need it:** To stop the "Midnight Fraud."
    *   *Scenario:* A Teller's credentials are used to access the system at 2:00 AM on a Sunday.
    *   *RBAC says:* "Allowed" (It's a Teller).
    *   *ABAC says:* "Denied" (Wrong Time).

**Illustration: RBAC vs ABAC**

```text
[RBAC Criteria]                 [ABAC Criteria]
+--------------+                +---------------------+
|              |                | Who? (Teller)       |
|  Role = CEO  |  ----->        | Where? (Office IP)  | ----> ACCESS GRANTED
|              |                | When? (Working Hrs) |
+--------------+                +---------------------+
   (Static)                            (Dynamic)
```

---

## 4.0 Credential Management and The Identity Crisis

### 4.1 Passwords are Dead (Long Live MFA)
Governance Policy: "Passwords alone are considered insufficient security."

### 4.2 Multi-Factor Authentication (MFA)
You need 2 out of 3:
1.  **Something you Know** (Password, PIN).
2.  **Something you Have** (Phone, Token, Smart Card).
3.  **Something you Are** (Fingerprint, Face ID).

### 4.3 The "SIM Swap" Vulnerability (West African Context)
This is the single biggest governance risk in our region regarding MFA.
*   **The Mechanism:**
    1.  The attacker identifies a target with a fat bank account.
    2.  They go to a Telco agent (or pay a corrupt insider) with a fake ID/Police Report claiming "My phone is lost."
    3.  A new SIM card is issued to the attacker. The victim's phone goes dead ("No Service").
    4.  All "One Time Passwords" (SMS OTPs) from the bank now go to the attacker.
    5.  They empty the account.

*   **Governance Defense:**
    *   **Stop using SMS for 2FA.**
    *   **Mandate App-Based Authenticators** (Google Auth, Microsoft Auth) which are tied to the *device*, not the SIM.
    *   **Biometric Verification:** Telcos now require live biometric verification for SIM replacement (The "I am Alive" check).

---

## 5.0 Comprehensive Case Studies

### Case Study 1: The "MoMo Fraud Ring" (African Context)

**Context:**
A major Telco in West Africa noticed a spike in "Reversals." (A Reversal is when an agent claims they sent money to the wrong number and ask customer service to reverse it).

**The Mechanism:**
*   **The Insider:** A corrupt supervisor in the Customer Support center had "Super Admin" privileges (RBAC failure).
*   **The Fraud:**
    1.  Syndicate sends 5,000 GHS to a co-conspirator.
    2.  Conspirator withdraws cash immediately.
    3.  Syndicate calls the Corrupt Supervisor.
    4.  Supervisor uses "Super Admin" tool to *Force Reverse* the transaction, pulling money back from the agent's float (creating a debt) or from the system suspense account.
    5.  They steal the money twice.

**The Governance Investigation:**
1.  **Access Control Failure:** Why did a Supervisor have the ability to reverse *settled* transactions without a second approver (Maker-Checker)?
2.  **Audit Trail Failure:** The logs showed the reversal, but no one was monitoring the "High Value Reversals" report.

**Remediation:**
Implementation of **ABAC**: Reversals > 1,000 GHS now require:
*   Approval from 2 Managers (Maker-Checker).
*   Biometric Authorization (Fingerprint of the Supervisor).

---

### Case Study 2: The S3 Bucket Leak (Global Context)

**Context:**
**Capital One (2019)** suffered a massive breach where 100 million credit card applications were exposed.

**The Mechanism:**
*   **The Misconfiguration:** A Firewall (WAF) was misconfigured, allowing a command to be sent to the backend server.
*   **The IAM Failure:** The server had an IAM Role (Identity Access Management) with *too much permission*. It was allowed to "List All Buckets" and "Read All Data."
*   **The Result:** The attacker just commanded the server to "Copy all data to my private server."

**Relevance to Africa:**
As our banks move to Cloud (Azure/AWS), this is the #1 risk. We are used to "Perimeter Security" (Firewalls). In the Cloud, **Identity is the Perimeter**. If you give a server "Admin" rights, you have breached yourself.

**Lesson:**
**Principle of Least Privilege.** A server should only be able to read the *specific* file it needs, not the whole database.

---

## 6.0 Module Assessment (Professional)

*Instructions: Select the most accurate answer.*

**Q1. In the context of a Ghanaian Telco, what is the primary security vulnerability associated with using SMS for 2-Factor Authentication?**
A) SMS is expensive.
B) SMS can be intercepted via SIM Swap attacks.
C) SMS requires internet.
D) SMS is too fast.

**Q2. Which Access Control model would best prevent a "Midnight Fraud" scenario where a user has the right Role but logs in at a suspicious time?**
A) RBAC (Role-Based).
B) ABAC (Attribute-Based).
C) MAC (Mandatory Access Control).
D) None.

**Q3. "Availability" in the CIA Triad, within an African infrastructure context, must primarily account for:**
A) Quantum Computing.
B) Power Instability (Dumsor) and Connectivity Redundancy.
C) GDPR compliance.
D) Software licensing.

**Q4. A "Maker-Checker" policy is a control for:**
A) Confidentiality.
B) Integrity (Preventing a single person from executing a high-risk action).
C) Availability.
D) Speed.

**Q5. Data Encryption "At Rest" protects against:**
A) A hacker intercepting the data while it is being emailed.
B) A thief stealing the physical hard drive / server.
C) A user guessing a password.
D) A power outage.

**Q6. What is the Principle of Least Privilege?**
A) Giving everyone Admin access so work is not blocked.
B) Giving a user/system only the minimum permissions necessary to do their job.
C) Giving privileges based on seniority (CEO gets everything).
D) Giving privileges based on who asks nicest.

**Q7. In the MoMo Fraud Case Study, what was the critical Governance failure?**
A) The internet was slow.
B) Lack of "Segregation of Duties" (Maker-Checker) for high-value reversals.
C) The agents were rude.
D) The phones were old.

**Q8. Which of the following is an example of "Something you Are" in Authentication?**
A) A Password.
B) A Smart Card.
C) A Fingerprint.
D) A PIN.

**Q9. Hashing (e.g., SHA-256) is primarily used to ensure:**
A) Confidentiality (Reading the message).
B) Data Integrity (Verifying the message hasn't changed).
C) Availability.
D) Speed.

**Q10. Why is "Data Masking" important for Customer Support Agents?**
A) It looks cool.
B) It reduces the "Insider Threat" risk by hiding sensitive PII (like full ID numbers) from employees who don't need to see it.
C) It saves ink when printing.
D) It makes the system faster.

---

## 7.0 Further Reading & Citations

1.  **Standard:** NIST SP 800-53 *Security and Privacy Controls*.
2.  **Concept:** *The CIA Triad*. (ISC)² CISSP Common Body of Knowledge.
3.  **Local Context:** *Bank of Ghana Cyber & Information Security Directive (2018)*.
4.  **Case Study:** *Capital One Data Breach Report*.

**End of Module 3.**
