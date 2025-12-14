# Module 4: Legal, Privacy, and User-Facing Documentation

**Instructor:** Ishmael Abayeteye, Data Engineer at AfriqueTek
**Course Level:** Executive / Professional Certification
**Duration:** 4 - 5 Hours
**Prerequisites:** Completion of Module 3.

---

## 1.0 Introduction: The "Legal Shield"

### 1.1 Welcome
Welcome to Module 4. We have Strategized (Mod 1), Managed (Mod 2), and Secured (Mod 3). Now, we must **Legitimize**.
In the data world, you can have High Quality, Secure data—and still go to jail. Why? Because you didn't have the *Legal Right* to collect it in the first place.

### 1.2 The Local Reality
In Ghana, the "Wild West" era of data is over.
*   **Past:** You could buy a USB drive full of phone numbers at Kwame Nkrumah Circle and send bulk SMS.
*   **Present:** The **Data Protection Commission (DPC)** is actively prosecuting non-compliant companies.
*   **The Risk:** It's not just the fine (which is measured in Penalty Units); it's the "Name and Shame" list published in the Daily Graphic.

**Module Learning Outcomes:**
1.  **Navigate:** Interpret the 8 Principles of the **Ghana Data Protection Act, 2012 (Act 843)**.
2.  **Draft:** Create a "Privacy Policy" and "Terms of Use" step-by-step using our practical workshop.
3.  **Contract:** Negotiate a **Data Processing Addendum (DPA)** with third-party vendors.
4.  **Manage:** Handle internal Employee Data privacy issues (BYOD).

---

## 2.0 Privacy Frameworks: The Rule of Law

### 2.1 The Global Standard: GDPR
The **General Data Protection Regulation (GDPR)** set the template for the world.
*   **Key Concept:** "Extraterritoriality." Even if you are a Ghanaian hotel, if you host a German tourist and email them a receipt, you are processing EU data.

### 2.2 The Local Standard: Act 843
Passed in 2012, this is our Bible. As a Data Governance professional, you must memorize specific sections.

#### The "Big Three" Compliance Checks:

1.  **Registration (Section 18):**
    *   *The Law:* "A person who processes personal data shall register with the Commission."
    *   *The Reality:* Many firms operate illegally. If you are not on the DPC public register, you are non-compliant.
    *   *Governance Action:* Check your certificate expiration date *today*.

2.  **Consent (Section 23):**
    *   *The Law:* Recent activity has clarified that consent must be **"Prior, Explicit, and Specific."**
    *   *The Trap:* Pre-ticked boxes. "Uncheck this box if you *don't* want emails." **(ILLEGAL)**. The user must actively tick "Yes."

3.  **Data Sovereignty (Section 18 & Guidelines):**
    *   *The Law:* Restrictions on transferring data to countries with "lower" protection standards.
    *   *Cloud Implication:* If you store data in an AWS region in a country without a data law, you might be liable.

**Table 4.1: GDPR vs Act 843 Comparison**

| Feature | GDPR (Europe) | Act 843 (Ghana) |
| :--- | :--- | :--- |
| **Data Subject Rights** | Right to be Forgotten, Right to Portability. | Right to Access (S.32), Right to Correct (S.33), Right to Cease Processing (S.39). |
| **Breach Notification** | STRICT: 72 Hours max. | FLEXIBLE: "As soon as reasonably practicable" (S.31), but courts interpret this strictly now. |
| **Sanctions** | €20M / 4% Turnover. | Up to 5,000 Penalty Units or 2 years jail (S.95). |
| **Officer** | Data Protection Officer (DPO) mandatory for some. | Data Protection Supervisor mandatory for ALL controllers (S.58). |

---

## 3.0 Practical Drafting Workshop: How to Write User Policies

*Instructor Note: In this section, we will use a fictional company, **"AgroMoni Ghana Ltd"** (an app lending money to cocoa farmers), to show you exactly how to write these documents.*

### 3.1 Document 1: The Privacy Policy
This is your **"Declaration of Operations."** It tells the user (and the DPC) exactly what you are doing with their data.

**The Golden Rule:** Be specific. "We collect data to improve service" is too vague and likely illegal.

#### Step 1: The "Collection" Clause
*   **Logic:** List exactly what you take.
*   **AgroMoni Example:**
    > **2. Information We Collect**
    > To provide our loan service, we collect:
    > *   **Identity Data:** Your full name, Ghana Card PIN (Verified via NIA), and Date of Birth.
    > *   **Financial Data:** Mobile Money transaction history (collected via API with your PIN approval).
    > *   **Location Data:** Your GPS coordinates (Verified via GhanaPostGPS) to confirm farm location.

#### Step 2: The "Purpose" Clause (Lawfulness)
*   **Logic:** Connect every data point to a reason (Act 843 Section 20).
*   **AgroMoni Example:**
    > **3. How We Use Your Data**
    > *   **Credit Scoring:** We use your MoMo history to calculate your repayment capacity.
    > *   **Fraud Prevention:** We use your GPS location to verify you are physically in Ghana.
    > *   **Legal Compliance:** We store your Ghana Card details to comply with Bank of Ghana KYC regulations.

#### Step 3: The "Sharing" Clause (Disclosure)
*   **Logic:** Who else sees this? This is where many companies get sued.
*   **AgroMoni Example:**
    > **4. Third-Party Sharing**
    > We respect your privacy. We strictly DO NOT sell your data to betting companies or marketers. However, we legally share data with:
    > *   **Credit Reference Bureaus (e.g., XDS Data):** To report loan defaults as required by law.
    > *   **Technical Providers:** Our SMS provider (Hubtel) to send you transaction alerts.

#### Step 4: The "Rights" Clause
*   **Logic:** Tell the user their power (Act 843 S.32-35).
*   **Global/Local Note:** In Europe, you add "Right to Erasure." In Ghana, be careful—you cannot erase a debtor's data.
*   **AgroMoni Example:**
    > **6. Your Rights**
    > Under Act 843, you have the right to request a copy of your data. You may ask to correct errors (e.g., wrong spelling). *Note: Requests to delete data while you have an active loan will be denied until liability is cleared.*

---

### 3.2 Document 2: Terms of Use (ToU)
This is your **"Contract."** It protects *you* from the User.

#### Step 1: Acceptable Use Policy (Bad Behavior)
*   **Logic:** What are they NOT allowed to do?
*   **AgroMoni Example:**
    > **5. Prohibited Conduct**
    > You agree NOT to:
    > *   Use the App to launder money or finance terrorism.
    > *   Attempt to reverse-engineer our credit algorithm.
    > *   Submit a fake Ghana Card or one belonging to another person (Identity Theft).

#### Step 2: Limitation of Liability (The Shield)
*   **Logic:** If the app crashes and they lose a deal, can they sue you?
*   **AgroMoni Example:**
    > **8. Limitation of Liability**
    > AgroMoni provides the service "As Is." While we strive for uptime, we are not liable for losses caused by:
    > *   Telco network failures (MTN/Telecel downtime).
    > *   Power outages affecting your device.
    > *   Delays in Mobile Money settlement.

#### Step 3: Termination (The Ban Hammer)
*   **Logic:** When can you kick them out?
*   **AgroMoni Example:**
    > **10. Account Termination**
    > We reserve the right to suspend your account immediately if we detect:
    > *   Suspicious login patterns (e.g., login from Nigeria while GPS says Kumasi).
    > *   Abusive language towards our support staff.

---

## 4.0 Vendor Management: The "Third Party" Risk

### 4.1 The Weakest Link
The DPC finds that 60% of breaches happen at the *Vendor* level (e.g., your Cloud Provider or Payroll Processor).

### 4.2 The Data Processing Addendum (DPA)
You must sign a DPA with every vendor. This is a contract that says:
1.  **Instruction:** "You (Vendor) only act on my instructions."
2.  **Security:** "You must maintain encryption."
3.  **Notification:** "If you are hacked, you must tell me within 24 hours."

**Case Scenario:** Your bank hires a "Cloud Payroll Provider."
*   **Without DPA:** If they leak data, they claim "We are just software." You pay the fine.
*   **With DPA:** They are contractually liable to indemnify you for the fines.

---

## 5.0 Employee Data Protection (Internal Privacy)

*Many companies forget that their own employees are Data Subjects too.*

### 5.1 Expectation of Privacy vs. Monitoring
*   **The Conflict:** Employers want to read emails to prevent fraud. Employees want privacy.
*   **The Governance Rule:** You can monitor, *if* you informed them first.
*   **Policy Text Example:**
    > "Company devices are for business use. The Company reserves the right to monitor email and internet traffic on corporate laptops. Employees should have no reasonable expectation of privacy on these devices."

### 5.2 BYOD (Bring Your Own Device) Policies
In Ghana, many staff use their personal phones for work WhatsApp groups.
*   **The Risk:** Staff leaves the company -> Takes the phone -> Still has the confidential client list on WhatsApp.
*   **The Governance Fix:**
    1.  **MDM Software:** Install "Mobile Device Management" (e.g., Microsoft Intune) that allows you to "Remote Wipe" *only* corporate data from their personal phone.
    2.  **The "WhatsApp" Policy:** Explicitly ban sending sensitive Excel sheets via personal WhatsApp. Use MS Teams instead.

---

## 6.0 Comprehensive Case Studies

### Case Study 1: The "Bulk SMS" Blunder (Local Context)
**Context:** **"ElectionWin 2024"** (fictional) purchased 50,000 numbers from a "Data Broker" at Circle to send political SMS: *"We know you vote at Station A."*
**The Violation:**
1.  **No Consent (S.20):** Citizens never agreed to share Polling Station data.
2.  **Direct Marketing (S.37):** Political spam is regulated.
**Result:** DPC Fines and PR disaster.
**Lesson:** **Data Provenance.** If you can't prove where the list came from, don't use it.

### Case Study 2: The "Hidden SDK" Trap (Global Context)
**Context:** A free Flashlight App used an SDK from an ad network.
**The Mechanism:** When the light turned on, the SDK uploaded **GPS Location** and **CONTACTS** to a server in China.
**The Legal Failure:** The Policy said *"We collect data to improve performance."* This is **Deceptive**. Selling data is not "performance improvement."
**Lesson:** **Transparency.** You must list the *real* purpose.

---

## 7.0 Module Assessment (Professional)

*Instructions: Select the most accurate answer.*

**Q1. According to Section 18 of Ghana's Act 843, what is the prerequisite to process data?**
A) Pay taxes.
B) Register with the DPC.
C) Have a website.
D) Be a multinational.

**Q2. When drafting a Privacy Policy for AgroMoni (a loan app), why is "We collect financial data" insufficient?**
A) It is too long.
B) It is too vague. Act 843 requires Specificity (e.g., "We collect MoMo Transaction History to score credit").
C) It is too specific.
D) It is perfect.

**Q3. Under the "Limitation of Liability" clause in the Terms of Use, a company typically disclaims responsibility for:**
A) Its own intentional fraud.
B) Third-party failures like Telco downtime or Power outages.
C) Everything.
D) Paying salaries.

**Q4. In a BYOD (Bring Your Own Device) policy, what is the primary security recommendation?**
A) Banning phones.
B) Using MDM software to separate/wipe corporate data without erasing personal photos.
C) Reading all employee texts.
D) Sharing phones.

**Q5. Why must you sign a DPA with your Payroll Vendor?**
A) To make them pay a fee.
B) To ensure they are legally liable if *they* leak your employees' salary data.
C) Because they are friends.
D) To get a discount.

**Q6. Which of the following is an "Acceptable Use" violation typical in FinTech?**
A) Logging in 3 times.
B) Using the app to launder money or financing terrorism.
C) Checking your balance.
D) Paying a loan early.

**Q7. The "Right to Access" (Section 32) allows a Data Subject to:**
A) Hack into your database.
B) Demand a copy of all the personal data you hold about them.
C) Demand free money.
D) Use your office wifi.

**Q8. If an employee uses a company laptop to send personal emails:**
A) They have total privacy.
B) They should have "No reasonable expectation of privacy" if the policy was clearly communicated beforehand.
C) They are fired immediately.
D) The laptop explodes.

**Q9. Why might a USSD-based app require a "Voice/IVR" Privacy Policy in rural Ghana?**
A) Farmers cannot read.
B) To ensure "Informed Consent" for users with low literacy.
C) It is cheaper.
D) DPC likes radio.

**Q10. What is the penalty for failure to register with the DPC under Act 843?**
A) Warning letter.
B) Up to 5000 Penalty Units or 2 years jail.
C) Nothing.
D) Loss of internet.

---

## 8.0 Further Reading & Citations
1.  **Legal Text:** *Republic of Ghana Data Protection Act, 2012 (Act 843)*.
2.  **Guide:** *Open Law Library - Drafting Privacy Policies*.
3.  **Global Comparison:** *GDPR Articles 12-14 (Transparency)*.

**End of Module 4.**
