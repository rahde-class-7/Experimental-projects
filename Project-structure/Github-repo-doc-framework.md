## [TEMPLATE] SEIR Project Documentation Framework
**File Name:** `README.md` (To be placed in each sub-directory)

---

### 1. Context & Business Logic
*This section answers: "Why does this exist in a real company?"*

* **Project Title:** [Clear, descriptive name]
* **Problem Statement:** [What technical gap or security risk are you solving?]
* **SEIR Domain:** [Networking / Identity / Data Protection / Automation]
* **Assumed Role:** [e.g., Cloud Security Architect / DevOps Engineer]

---

### 2. Technical Architecture
*The "blueprint" of the solution.*

* **Architecture Diagram:** `![Alt Text](link_to_image)`
* **Resource Inventory:** A table listing the GCP resources used (VMs, VPCs, IAM Roles, Buckets).
* **Decision Log:** Why did you choose Service Accounts over User Accounts? Why this specific CIDR?

---

### 3. Security & Identity Controls (The "IR" in SEIR)
*This is the core of your specialization. Do not skip.*

* **Identity Principle:** [e.g., Used Least Privilege by creating custom IAM roles].
* **Network Perimeter:** [e.g., Isolated DB via Private Google Access].
* **Data Protection:** [e.g., Encrypted at rest using CMEK or protected by VPC-SC].
* **Auditability:** [e.g., Enabled Cloud Audit Logs for all administrative writes].

---

### 4. Deployment Instructions
*Can another engineer (or your future self) recreate this in 5 minutes?*

* **Prerequisites:** [e.g., Enable Compute Engine API, Install gcloud SDK].
* **Execution:**
    * **Option A (CLI):** Provide `gcloud` code blocks.
    * **Option B (IaC):** Provide links to `.tf` (Terraform) files in the directory.
* **Configuration Quirks:** [Any "gotchas," like waiting 5 minutes for a Global Load Balancer to propagate].

---

### 5. Verification & Quality Assurance (The "Proof")
*Don't just say it works. Show it.*

* **Test Case 1:** [Describe test, e.g., "Internal connectivity check"] -> **Result:** [Pass/Fail].
* **Test Case 2:** [Describe security test, e.g., "Attempted unauthorized access"] -> **Result:** [Blocked].
* **Logs/Evidence:** Markdown blockquote of a successful command output or a screenshot.

---

### 6. Post-Mortem & Engineering Growth
*This shows you have a "Senior Engineer" mindset.*

* **What went wrong?** [Be honest. Troubleshooting is a skill].
* **Future Improvements:** [e.g., "Next time, I would automate the firewall rules using a CI/CD pipeline"].
* **Certification Alignment:** [Which exam objective did this satisfy? PCA, PCSE, etc.]

---

## 💡 SEIR Writing Standards (Best Practices)

| **Do** | **Don't** |
| :--- | :--- |
| Use **Passive/Professional** tone ("The VPC was created...") | Use **Informal** tone ("I made a VPC...") |
| Use **Table formats** for data comparison. | Use **Long paragraphs** for technical specs. |
| Mask **Sensitive Info** (Project IDs, Keys). | Leave **Project IDs** or **IPs** in screenshots. |
| Focus on **Architecture & Logic**. | Focus only on **Button Clicking**. |

---

### **How to use this Reference Guide:**
1.  Keep this file as `REFERENCE_GUIDE.md` in your repo root.
2.  Whenever you start a new lab (e.g., **Identity-Aware Proxy** or **Cloud KMS**), copy this text into a new `README.md` in that folder.
3.  Fill in the brackets `[ ]` with your specific project details.

Does this framework feel like a solid foundation for your future GCP projects, or would you like to add a section for **Budget/Cost Optimization** (a common PCA topic)?
