# Module 5: Incident Handling and Response

**Instructor:** Ishmael Abayeteye, Data Engineer at AfriqueTek
**Course Level:** Executive / Professional Certification
**Duration:** 3 - 4 Hours
**Prerequisites:** Completion of Module 4.

---

## 1.0 Introduction: "Not IF, but WHEN"

### 1.1 Welcome
Welcome to the final module. We have built the castle (Governance), cleaned the rooms (Quality), locked the gates (Security), and signed the laws (Legal).
But what happens when the enemy breaches the wall?

In cybersecurity, we have a saying: *There are two types of companies: those who have been hacked, and those who don't know they have been hacked.*

### 1.2 The Local Reality
In West Africa, our approach to incidents is often "Silence and Prayer."
*   **The Bad Practice:** A bank gets hacked. They hide it. They pay the ransom. They don't patch the hole. Two months later, they get hacked again.
*   **The Professional Approach:** We treat an incident as a **Governance Failure** that requires a structured, scientific response.
*   **Unique Challenge:** **Cyber Insurance** is rare in Ghana. If you get hit with Ransomware demanding $1 Million Bitcoin, you are paying out of pocket (or liquidating assets).

**Module Learning Outcomes:**
1.  **Execute:** Navigate the 6 Phases of the **Incident Response (IR) Cycle (PICERL)**.
2.  **Comply:** strict adherence to **Section 31** of Act 843 regarding Breach Notification.
3.  **Recover:** Design a "Business Continuity Plan" (BCP) that works even during "Dumsor" (Power outages).

---

## 2.0 The Incident Response (IR) Plan: PICERL In-Depth

We follow the **NIST (National Institute of Standards and Technology)** framework. The acronym is **PICERL**.
**P**reparation -> **I**dentification -> **C**ontainment -> **E**radication -> **R**ecovery -> **L**essons Learned.

We will now dissect every single stage.

---

### Phase 1: Preparation (The "Fire Drill" Stage)

**"The more you sweat in training, the less you bleed in battle."**
This is the only phase that happens *before* the hack. If you fail here, you fail everywhere.

#### Key Terms Defined:
*   **CSIRT (Computer Security Incident Response Team):** This is your specialized "SWAT Team." It is NOT just IT people. It includes Legal (to handle Act 843), HR (to manage internal leaks), and PR (to handle the media).
*   **Air-Gapped Backup:** A backup copy that is physically disconnected from the network. If your main server gets Ransomware, the virus cannot crawl across the wire to infect the air-gapped tape/drive.
*   **Tabletop Exercise:** A simulation where the team sits in a room and "pretends" a hack is happening to test if they know who to call.

#### Action Checklist (What to do NOW):
1.  [ ] **Appoint the CSIRT:** Write down names and phone numbers. Who is the "Incident Commander"?
2.  [ ] **Create "Runbooks":** Step-by-step guides for specific attacks. (e.g., "If Phishing, do steps 1-5").
3.  [ ] **Establish Out-of-Band Communication:** If the hackers control your company Email (Outlook), how will you talk? (Set up a secure Signal/WhatsApp group).

#### African Context:
In Ghana, "Preparation" also means Power. Does your Server Room have a UPS (Battery) that lasts 15 minutes, or a Diesel Generator that lasts 24 hours? If you are mitigating an attack and "Dumsor" (lights off) happens, you lose.

---

### Phase 2: Identification (The "Alarm" Stage)

This is the moment you realize something is wrong.

#### Key Terms Defined:
*   **SIEM (Security Information and Event Management):** Software (like Splunk or AlienVault) that collects logs from all your computers and looks for patterns. It screams "Alert!" if it sees weird behavior.
*   **Dwell Time:** The time the hacker sits inside your network *before* you find them. In Africa, the average is **200+ days**.
*   **False Positive:** An alarm that is wrong (e.g., The CEO forgot his password 5 times, and the system thought it was a hacker).

#### Action Checklist:
1.  [ ] **Monitor Logs:** Check your firewall and server logs daily.
2.  [ ] **Validate the Alert:** Is it real? Verify before you panic.
3.  [ ] **Declare the Incident:** The Incident Commander officially says "We are under attack." This starts the clock for Act 843 reporting.

#### African Context:
Often, the first sign of a breach in our region is not a siren, but a customer complaining on **Twitter/X**: *"My money has vanished!"* Monitoring Social Media is part of Identification.

---

### Phase 3: Containment (The "Stop the Bleeding" Stage)

You found the hacker. Now, stop them from moving.

#### Key Terms Defined:
*   **Lateral Movement:** The hacker moving from one computer (Receptionist) to another (CEO) to find the "Crown Jewels."
*   **Short-term Containment:** Stopping the immediate threat (e.g., Unplugging the internet cable).
*   **Long-term Containment:** Applying patches so they can't get back in the same way.

#### Action Checklist:
1.  [ ] **Isolate the Host:** Take the infected laptop off the WiFi.
2.  [ ] **Reset Credentials:** Force a password reset for ALL Admin users.
3.  [ ] **Update Firewalls:** Block the IP address the hacker is coming from.
4.  [ ] **Decision Point:** Do we shut down the business (Website) to save the data? (This is a CEO decision).

#### African Context:
**Ransomware Defense.** In Ghana, we see "Double Extortion." The hacker encrypts your files AND steals them to blackmail you. **Containment Rule:** If you see Ransomware, **Disconnect IMMEDIATELY**. Do not try to "save your work." Pull the plug.

---

### Phase 4: Eradication (The "Clean Up" Stage)

The hacker is blocked. Now we must remove the infection.

#### Key Terms Defined:
*   **Root Cause Analysis (RCA):** Finding *exactly* how they got in. Was it a weak password? A phishing email? An unpatched server?
*   **Golden Image:** A "perfect," clean copy of your operating system that you use to rebuild servers. Never try to "clean" a virus manually; wipe the machine and reload the Golden Image.

#### Action Checklist:
1.  [ ] **Re-image Systems:** Format the hard drives of infected machines.
2.  [ ] **Patch Vulnerabilities:** If they got in via a "Windows Bug," install the Windows Update patch.
3.  [ ] **Scan for Backdoors:** Hackers leave "secret keys" to get back in. Scan specifically for these.

---

### Phase 5: Recovery (The "Business Usual" Stage)

Bringing the systems back online.

#### Key Terms Defined:
*   **RPO (Recovery Point Objective):** How much data can we afford to lose? (e.g., "We can lose 1 hour of data").
*   **RTO (Recovery Time Objective):** How fast must we be back up? (e.g., "We must be up in 4 hours").

#### Action Checklist:
1.  [ ] **Restore from Clean Backups:** Use those Air-gapped backups from Phase 1.
2.  [ ] **Phased Rollout:** Don't turn everything on at once. Turn on HR first, verify. Turn on Sales, verify.
3.  [ ] **Enhanced Monitoring:** Watch the network like a hawk for 48 hours. The hacker might still be hiding.

#### African Context:
**Connectivity Constraints.** Downloading 50 Terabytes of data from the Cloud (AWS Ireland) to a server in Accra might take **weeks** over standard internet. This is why "Local Backups" (On-premise tapes) are critical for fast recovery.

---

### Phase 6: Lessons Learned (The "Wisdom" Stage)

Two weeks later. The dust has settled.

#### Key Terms Defined:
*   **Post-Mortem Report:** A formal document analyzing the incident.
*   **Governance Update:** Changing company policy based on what happened.

#### Action Checklist:
1.  [ ] **Draft the Report:** Executive Summary (for CEO) + Technical Detail (for IT).
2.  [ ] **Update the Budget:** "We need to buy a better Firewall."
3.  [ ] **Retrain Staff:** If it was a Phishing attack, run a training session for all employees.

---

## 3.0 Breach Notification: The Law (Act 843)

### 3.1 To Tell or Not To Tell?
Most CEOs want to keep quiet to protect the stock price. As the Data Governance Officer, you must tell them: **"Silence is Illegal."**

### 3.2 Act 843 Section 31
*   **The Obligation:** "Where there are reasonable grounds to believe that... personal data has been accessed... the Data Controller shall notify the Commission and the Date Subject."
*   **The Timeline:** "As soon as reasonably practicable."
    *   *Note:* The DPC interprets this aggressively. Delaying for 2 weeks is unacceptable.
*   **The Exception:** You don't have to notify the *Subject* if the data was encrypted (unreadable to the thief). But you must always notify the *Commission*.

**Illustration: The Notification Decision Tree**

```text
[Breach Detected]
       |
[Does it involve Personal Data?] --NO--> [Internal IT Issue. Fix it.]
       |
      YES
       |
[Was the Data Encrypted?] --YES--> [Notify Regulator (DPC). No need to panic customers.]
       |
       NO (Plain text)
       |
[CRITICAL ALERT]
1. Notify DPC immediately.
2. Notify Customers (Email/SMS) "Your data was exposed."
3. Prepare PR Statement.
```

---

## 4.0 Comprehensive Case Studies

### Case Study 1: Ransomware in the Local Power Grid (African Context)

**Context:**
**"WestPower Co"** (fictional utility) manages the prepaid meter billing system.

**The Incident:**
On a Friday night, the IT manager receives a call. The billing server displays a red screen: *"YOUR FILES ARE ENCRYPTED. PAY 50 BTC."*
Panic ensues. Customers cannot buy credit. Lights start going off across the capital.

**The Response (Pass/Fail):**
*   **Identification (FAIL):** They ignored anti-virus alerts for 2 weeks.
*   **Containment (PASS):** They physically pulled the ethernet cables to save the Power Plant control systems (SCADA).
*   **Recovery (FAIL):** They tried to restore from backup, but the backups were on a hard drive *connected* to the main server. The Ransomware encrypted the backups too.
*   **Decision:** They had to rebuild the database from paper receipts, taking 3 months.

**The Governance Lesson:**
**Immutable Offsite Backups.** The "3-2-1" rule (3 copies, 2 media, 1 offsite) is the *only* defense against Ransomware. If your backup is connected, your backup is dead.

---

### Case Study 2: The Supply Chain "Trojan Horse" (Global Context)

**Context:**
**SolarWinds (2020)**. A software used by the US Government and Fortune 500s to manage networks.

**The Mechanism:**
Hackers compromised SolarWinds' *build system*. They inserted malware into a legitimate software update.
*   **The Infection:** 18,000 companies downloaded the "Security Update" which actually contained the backdoor.
*   **The Stealth:** The hackers sat quiet for 9 months (Identification failure).

**Relevance to You:**
You trust your vendors (Banking Software, HR App). What if *they* are the virus?
*   **Governance Action:** **Network Segmentation.** Do not give your vendor's software unlimited access to your entire network. Put it in a box (VLAN).

---

## 5.0 Interactive Tabletop Exercise

*Instructor Note: In a live class, I would run this as a role-playing game. For this document, think through your answer.*

**Scenario:**
You are the Head of Data at a Fintech in Accra. It is 4:45 PM on Friday.
Your Lead Engineer walks in, pale-faced. "Boss, I found a 10GB file on our server being uploaded to an IP address in Ukraine. It looks like the full customer database."

**Decision Point 1:**
Do you shut down the server immediately?
*   *Option A:* Yes. Stop the leak.
*   *Option B:* No. Watch and see what else they take.
*   *Correct Strategy:* **Option A**. In a data theft scenario involving PII, stopping the exfiltration is priority #1 to minimize Act 843 liability.

**Decision Point 2:**
It is now 6:00 PM. The leak is stopped. Who do you call first?
*   *Option A:* The Police.
*   *Option B:* The CEO and Legal Counsel.
*   *Option C:* Determining if the data was encrypted.
*   *Correct Strategy:* **Option C, then B.** If it was encrypted, the crisis is small. If not, the crisis is existential.

---

## 6.0 Module Assessment (Professional)

*Instructions: Select the most accurate answer.*

**Q1. What does the acronym "PICERL" stand for in Incident Response?**
A) Plan, Ignore, Cry, Erase, Run, Lie.
B) Preparation, Identification, Containment, Eradication, Recovery, Lessons Learned.
C) Prevention, Identification, Clean, Erase, Recover, Log.
D) Policies, Incidents, Controls, Errors, Risks, Laws.

**Q2. Under Act 843 Section 31, usually, when must you notify the Data Protection Commission of a breach?**
A) Within 1 hour.
B) As soon as reasonably practicable.
C) After you fix it.
D) Never.

**Q3. In the "WestPower" Ransomware case, why did the Recovery fail?**
A) The hackers were too good.
B) The backups were connected to the network and got encrypted too (Lack of Air-gap).
C) The internet was off.
D) They didn't have Bitcoin.

**Q4. Why is "Preparation" considered the most important phase of IR?**
A) Because it is the cheapest.
B) Because once the hack starts, it is too late to buy fire extinguishers (Backups/Tools).
C) Because NIST says so.
D) It is not important.

**Q5. What is "Dwell Time"?**
A) The time it takes to hack a password.
B) The duration an attacker remains undetected inside a network (often months).
C) The time to write a report.
D) The time spent in jail.

**Q6. During the "Containment" phase of a Ransomware attack, what is the standard recommended action?**
A) Pay the ransom immediately.
B) Disconnect infected systems from the network to prevent spread.
C) Email the hacker.
D) Reboot the server (which might delete evidence).

**Q7. If a breach involves encrypted data (where the key was NOT stolen), are you legally required to notify the Data Subject (Customer) under most frameworks?**
A) Yes, always.
B) No, because the risk of harm is low (they cannot read the data).
C) Only if they are VIPs.
D) Yes, but only via SMS.

**Q8. The "SolarWinds" attack demonstrated the risk of:**
A) Phishing emails.
B) Supply Chain Attacks (Compromised trusted vendors).
C) Weak passwords.
D) Unlocked doors.

**Q9. Who should be a member of the CSIRT (Incident Response Team)?**
A) Only IT staff.
B) Cross-functional members including Legal, HR, PR, and Management.
C) The Police only.
D) The Intern.

**Q10. What is the purpose of the "Lessons Learned" phase?**
A) To find someone to blame and fire.
B) To analyze the root cause and improve defenses so it doesn't happen again.
C) To write a press release.
D) To forget the incident.

---

## 7.0 Further Reading & Citations

1.  **Standard:** NIST SP 800-61 *Computer Security Incident Handling Guide*.
2.  **Legal:** *Ghana Data Protection Act, 2012 (Section 31)*.
3.  **Global:** *ISO/IEC 27035: Information Security Incident Management*.

**End of Module 5.**
**Course Complete.**
