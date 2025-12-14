# Module 2: Core Data Lifecycle Policies and Quality

**Instructor:** Ishmael Abayeteye, Data Engineer at AfriqueTek
**Course Level:** Executive / Professional Certification
**Duration:** 3 - 4 Hours
**Prerequisites:** Completion of Module 1.

---

## 1.0 Introduction: "Garbage In, Disaster Out"

### 1.1 Welcome
Welcome to Module 2. In Module 1, we set the *laws* (Governance). Now, we must ensure the *soil* is fertile.
As a Data Engineer, I spend 60-70% of my time cleaning data. Why? Because organizations treat data entry as a low-priority task, only to realize years later that their database is full of "ghosts," duplicates, and incomplete records.

### 1.2 The Core Philosophy
*   **The Global Standard:** **ISO 8000** is the international standard for Data Quality. It defines quality not as "perfect," but as "fit for purpose."
*   **The Local Reality:** In our context, "fitness" is harder to achieve. We deal with inconsistent street addresses, varying name spellings due to transliteration (e.g., *Mohammed* vs *Muhammed*), and legacy paper records being digitized.

**Module Learning Outcomes:**
1.  **Assess:** Audit a dataset using the 6 Core Dimensions of Data Quality.
2.  **Architect:** Design a high-level conceptual data model that respects local infrastructure constraints (e.g., internet latency).
3.  **Retain:** Build a Data Retention Schedule that complies with both business needs and Ghanaian legal requirements.

---

## 2.0 Data Quality Management: The Health of Your Assets

### 2.1 The Six Dimensions of Data Quality (DAMA-DMBOK)
You cannot manage what you cannot measure. We use six yardsticks to measure data health.

#### 1. Accuracy
*   **Definition:** Does the data reflect reality?
*   **Global Standard:** A customer's age is calculated from their DOB.
*   **Local Challenge:** In rural digitization projects, I have observed many "January 1st" birthdays. This is often a default value entered by agents when an elderly person does not know their exact DOB. If you analyze this, you will find a statistical anomaly.

#### 2. Completeness
*   **Definition:** Is all the required data present?
*   **Local Challenge:** "Residential Address." Before **GhanaPostGPS**, it was common to see "Near the Blue Kiosk, Techiman" as an address. This is *incomplete* for a logistics drone, even if it makes sense to a human taxi driver.

#### 3. Consistency
*   **Definition:** Is the data the same across all systems?
*   **Scenario:**
    *   *HR System:* Employee ID = `GH-001`
    *   *Payroll System:* Employee ID = `001`
    *   *Result:* The merge fails.

#### 4. Timeliness (Latency)
*   **Definition:** Is the data available when you need it?
*   **Local Context:** In Mobile Money (MoMo) fraud detection, data must be real-time (<2 seconds). If the data arrives 5 hours later (batch processing), the fraudster has already cashed out.

#### 5. Uniqueness
*   **Definition:** Is this entity recorded only once? (No duplicates).
*   **Local Context:** The "Name Variation" problem. *Osei-Tutu* vs *Osei Tutu*.

#### 6. Validity
*   **Definition:** Does the data follow the defined format rules?
*   **Global Standard:** Phone numbers must follow E.164 format.
*   **Local Standard:** Ghanaian numbers must match `+233 XX XXX XXXX`. A number starting with `024` is valid locally but invalid internationally without the prefix.

**Table 2.1: Data Quality Assessment Matrix**

| Dimension | Fails If... | Ghana/Africa Example |
| :--- | :--- | :--- |
| **Accuracy** | The data is false. | A customer is listed as "Male" but is receiving Maternity benefits. |
| **Completeness** | Fields are empty (Null). | A loan application lacks the guarantor's Ghana Card PIN. |
| **Consistency** | Systems disagree. | Marketing says we have 500 customers; Sales says 450. |
| **Timeliness** | Data is old. | A Credit Reference Bureau (CRB) provides a credit score based on data from 2018. |
| **Uniqueness** | Duplicates exist. | "Kwame Mensah" appears 4 times in the database. |
| **Validity** | Wrong format. | Entering "Accra" in the "Date of Birth" field. |

### 2.2 The "Address" Revolution: GhanaPostGPS
Nothing has impacted Ghanaian Data Quality more than the **National Digital Property Addressing System (NDPAS)**.

*   **Pre-2017:** Address data was unstructured text.
    *   *Example:* "P.O. Box 45, Opposite the Methodist Church."
    *   *Data Quality implication:* Impossible to plot on a map or analyze for logistics.
*   **Post-2017:** Structured Alphanumeric Codes.
    *   *Example:* `GA-183-8164`.
*   **Governance Mandate:** As a Data Manager, you must enforce a policy that **all** new customer onboarding forms *must* validate the GPS address against the official API. You stop accepting "Blue Kiosk" descriptions.

---

## 3.0 Data Architecture and Modeling: The Blueprints

### 3.1 Understanding Data Architecture
If Governance is the law, Architecture is the city planning. It decides where the pipes (data flows) go and where the reservoirs (databases) sit.

### 3.2 Conceptual vs. Logical vs. Physical
You must be able to speak to business leaders and techies differently. The biggest failure I see is a Data Architect showing a "Physical Schema" to a CEO. The CEO does not care about `VARCHAR(255)`; they care about "Customers".

**Table 2.2: The Three Layers of Data Modeling**

| Feature | **1. Conceptual** (The Business View) | **2. Logical** (The Architect View) | **3. Physical** (The Technical View) |
| :--- | :--- | :--- | :--- |
| **Audience** | Executives (CEO, Marketing) | Business Analysts, Data Architects | DBAs, Cloud Engineers |
| **Goal** | Understand business concepts. | Define rules and relationships. | Optimize performance and storage. |
| **Tech Details** | None. | Keys (PK/FK), Attributes. | Column Types, Indexes, Partitions. |

**Illustration: The Evolution of a "Customer" Model**

```text
[1. CONCEPTUAL]             [2. LOGICAL]                 [3. PHYSICAL]
+----------+                +----------------+           CREATE TABLE Cust_01 (
| Customer | --buys-->      |  Customer      |             cust_id INT PRIMARY KEY,
+----------+      |         |----------------|             f_name VARCHAR(50),
                  |         | PK: Cust_ID    |             l_name VARCHAR(50)
              +-------+     | FK: Region_Code|           ) TABLESPACE: aws_rds_01;
              | Order |     +----------------+
              +-------+
```

### 3.3 Infrastructure Realities: Cloud vs. On-Premise
In the West, the default is "Cloud First" (AWS/Azure). In our region, we must consider **Data Sovereignty** and **Connectivity**.

*   **The Connectivity Constraint:** While fiber is growing (MainOne, ACE cables), fiber cuts are common.
    *   *Architectural Decision:* If you are building a POS system for a supermarket in a rural district, a "Pure Cloud" architecture is risky. If the internet cuts, sales stop.
    *   *Solution:* **Edge Computing**. Have a local server (On-Prem) that handles sales and syncs to the Cloud (Head Office) when the internet returns. This is an architectural choice driven by local reality.

*   **The Sovereignty Constraint:**
    *   Some sectors (Banking, Government) are legally restricted from hosting "Core Banking Data" outside the jurisdiction of Ghana.
    *   *Governance Rule:* You cannot just spin up an AWS server in Ireland for the Ministry of Finance's payroll data without specific regulatory clearance.

---

## 4.0 Data Retention Schedule: The Lifecycle Policy

### 4.1 The Data Lifecycle In-Depth
Data is born, it lives, and it must die. Keeping data forever is expensive and risky (Toxic Data).
Most organizations in Ghana are great at **Collecting** and **Storing**, but terrible at **Archiving** and **Destroying**.

**The Cycle:** `Collect` -> `Store` -> `Use` -> `Share` -> `Archive` -> `Destroy`.

Let us break down the Governance implications of every single stage using a structured approach.

#### Stage 1: Collect (Creation)
This is where data enters your organization.
*   **The Governance Rule:** "Data Minimization." Only collect what you strictly need.
*   **Local Context:** Don't ask for a "Zip Code" in Ghana; we don't have them. Ask for "GhanaPostGPS."

**Table 2.3: Collection Stage Governance**

| Feature | Governance Check | Example (Loan App) |
| :--- | :--- | :--- |
| **Validation** | Is the data valid at entry? | Check if phone number is `+233...` |
| **Consent** | Did they say yes? (Act 843 S.23) | User ticks "I agree to credit check." |
| **Source** | Where did it come from? | "User Input" vs "Purchased List." |

#### Stage 2: Store (Maintenance)
The data is now sitting in your database.
*   **The Governance Rule:** Security and Sovereignty.
*   **Key Question:** Is this database in Accra (Local) or Ireland (Cloud)? If it is personal data, is it encrypted?

**Table 2.4: Storage Risk Matrix**

| Risk | Control Mechanism |
| :--- | :--- |
| **Theft** | Encryption at Rest (AES-256). |
| **Loss** | 3-2-1 Backup Strategy (3 copies, 2 media types, 1 offsite). |
| **Corruption** | Checksums and data integrity monitoring. |

#### Stage 3: Use (Processing)
This is the "Active" stage where business value is generated (Analytics, Reporting, App usage).
*   **The Governance Rule:** "Purpose Limitation."
*   **Scenario:** You collected phone numbers for "Account Security" (2FA).
*   **Violation:** Marketing uses that list to send SMS adverts for betting. This is illegal under Act 843 Section 20.

#### Stage 4: Share (Distribution)
Data leaving your secure walls.
*   **The Governance Rule:** Data Processing Addendums (DPA).
*   **Local Context:** Sharing data with a Credit Reference Bureau (e.g., XDS Data or Dun & Bradstreet). You *must* have a contract that says they cannot resell the data.

**Illustration: The Secure Sharing Flow**
```text
[Your Bank]  --1. Encrypt (PGP)--> [Secure FTP] --2. Download & Decrypt--> [Credit Bureau]
    ^                                                                           |
    |_______________________3. Contract (DPA) Protects This_____________________|
```

#### Stage 5: Archive (Long-term Storage)
The data is no longer active but must be kept for legal reasons.
*   **The Governance Rule:** Cost Reduction. Move data from expensive SSDs to cheap "Cold Storage" (Tape or AWS Glacier).
*   **Timeframe:** Usually 6-7 years (Tax laws).

#### Stage 6: Destroy (Purging)
The end of the road.
*   **The Governance Rule:** "No Trace Left Behind."
*   **Methodology:**
    *   *Digital:* Crypto-shredding (Deleting the encryption key, making data unreadable).
    *   *Physical:* Degaussing hard drives or shredding paper records.

**Table 2.5: The Destruction Protocol**

| Method | When to use | Security Level |
| :--- | :--- | :--- |
| **Simple Delete** | Never for sensitive data. | Low (Recoverable). |
| **Wipe / Overwrite** | Standard PCs. | Medium. |
| **Physical Destroy** | Failed Hard Drives. | High (Hammer/Shredder). |

### 4.2 Creating a Retention Schedule
A Retention Schedule is a policy document that tells you exactly how long to keep specific types of data. It is driven by **Legal Requirements** first, and **Business Needs** second.

**Table 2.6: Sample Retention Schedule (Ghana Context)**

| Data Type | Retention Period | Reason / Citation |
| :--- | :--- | :--- |
| **Employee Contracts** | 6 Years after termination | **Limitations Act, 1972 (NRCD 54**): Contract claims limitation period. |
| **Tax Records** | 6 Years | **Revenue Administration Act, 2016 (Act 915)**: GRA audit window. |
| **Call Data Records (Telco)** | Varies (Typically 1-2 Years) | **Electronic Communications Act**: For security and billing disputes. |
| **CCTV Footage** | 30 Days | Practical storage limits vs Security needs. |
| **Marketing Leads** | 1 Year | **Act 843 Principle**: Data Minimization. If they haven't bought in a year, delete them. |

### 4.3 The "Right to be Forgotten" (Erasure)
Under **Act 843**, a Data Subject can request the deletion of their data.
*   *The Conflict:* A customer who owes you money asks to be "forgotten."
*   *The Resolution:* You **deny** the request. Why? Because the "lawful purpose" (contract fulfillment/debt collection) overrides their consent. You only delete when the debt is paid + the retention period (6 years) has passed.

---

## 5.0 Comprehensive Case Studies

### Case Study 1: The "Ghost Pensioners" (African Context)

**Context:**
The Controller and Accountant General's Department (CAGD) manages the payroll for government employees and pensioners.

**The Narrative:**
For decades, the system relied on manual inputs from district offices.
*   **The Quality Failures:**
    *   *Uniqueness:* Deceased pensioners were not removed (No "Death Registry" integration).
    *   *Validity:* Fake names were inserted by corrupt officers to divert funds.
    *   *Accuracy:* "Ghost Names" accounted for millions of Cedis in losses monthly.

**The Solution (Data Governance Intervention):**
The government mandated a **Data Quality Cleansing** project tied to Biometric Verification (SSNIT/Ghana Card).
1.  **Policy:** "No Biometric Match = No Salary."
2.  **Execution:** All 500,000+ staff had to physically verify their biometrics.
3.  **Result:** Thousands of "Ghosts" vanished. The payroll bill dropped significantly.

**Analysis:**
This was not a technology problem (The payroll software worked fine). It was a **Veracity** problem. The data entering the system was corrupt.

---

### Case Study 2: The Retail Inventory Disaster (Global Context)

**Context:**
**Target Canada (2013-2015)** failed spectacularly, losing billions and shutting down in less than 2 years.

**The Narrative:**
When Target expanded from the US to Canada, they had to build a new supply chain system.
*   **The Rush:** They manually entered data for 75,000 products in weeks to meet a deadline.
*   **The Quality Failure (Dimensions):**
    *   *Accuracy:* Dimensions (Length/Width) were entered in *Inches* (US standard) but the system was configured for *Centimeters* (Metric).
    *   *Result:* The software thought the products were huge. It calculated that a shipping container could only hold 100 items instead of 1000.
    *   *Outcome:* Warehouses were overflowing (real world), but the computer said they were full (virtual world). Shelves were empty.

**Lesson for the Data Engineer:**
**Metadata Management** matters. Defining the "Unit of Measure" is a Governance standard. If you mix Imperial and Metric systems without conversion, you destroy the supply chain.

---

## 6.0 Module Assessment (Professional)

*Instructions: Select the most accurate answer.*

**Q1. You are designing a database for a logistics company in Kumasi. You find that 40% of the addresses are descriptive (e.g., "Behind the market"). Which Quality Dimension is most compromised?**
A) Timeliness.
B) Completeness (Structural).
C) Security.
D) Latency.

**Q2. What is the "GhanaPostGPS" code (e.g., GA-183-8164) primarily used for in a Data Quality context?**
A) To track user movements.
B) As a standardized, validatable attribute to resolve the "Address Ambiguity" problem.
C) To calculate taxes.
D) To generate random passwords.

**Q3. Under the Limitations Act (NRCD 54) of Ghana, what is the prudent retention period for contract-related financial documents?**
A) 1 Year.
B) 6 Years.
C) Forever.
D) Until the CEO retires.

**Q4. A "Ghost Name" in a payroll system is primarily a failure of which Data Quality dimension?**
A) Accuracy (Does the employee actually exist?).
B) Latency.
C) Format.
D) Encryption.

**Q5. Why might a rural bank in the Northern Region prefer an "Edge Computing" (Hybrid) architecture over a "Pure Cloud" architecture?**
A) Cloud is illegal.
B) To mitigate service downtime caused by internet connectivity cuts (Fiber cuts).
C) Servers are cheaper than Cloud.
D) Edge computing uses less electricity.

**Q6. Target Canada failed because they mixed up Inches and Centimeters. This is a failure of:**
A) Metadata Management (Context of data).
B) Data Security.
C) Data Privacy.
D) Network Speed.

**Q7. Which of the following is a valid reason to DENY a "Right to Erasure" request under Act 843?**
A) "We are too busy to delete it."
B) "The data is needed to comply with a legal obligation (e.g., Tax Law)."
C) "We like the customer."
D) "The delete button is broken."

**Q8. Constructing a "Conceptual Data Model" is best suited for which audience?**
A) The Database Administrator writing SQL.
B) The Business Stakeholders (CEO, Managers) to understand relationships.
C) The Hacker.
D) The Server Technician.

**Q9. "Data Profiling" is the process of:**
A) Stalking employees on social media.
B) Analyzing the existing data to determine its actual quality state (e.g., finding that 50% of email fields are empty).
C) Encrypting data.
D) Deleting data.

**Q10. In a "Federated" Data Quality model:**
A) The Head Office fixes all errors.
B) Each local department is responsible for the quality of the data they create/own.
C) No one is responsible.
D) The government fixes the data.

---

## 7.0 Further Reading & Citations

1.  **Global Standard:** ISO 8000 *Data Quality Standards*.
2.  **Local Law:** *Revenue Administration Act, 2016 (Act 915)* - for Retention rules.
3.  **Local Assessment:** *National Digital Property Addressing System (NDPAS)* technical documentation.
4.  **Case Analysis:** *Target Canada: The supply chain disaster*. Harvard Business Review case collection.

**End of Module 2.**
