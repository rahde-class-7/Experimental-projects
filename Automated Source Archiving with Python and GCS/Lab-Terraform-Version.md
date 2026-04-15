This lab is designed to move you away from "ClickOps" and into the mindset of a **Cloud Engineer**. In a production environment, manual configuration is a liability. You will build an automated, auditable, and repeatable source-archiving pipeline using **Terraform**.

---

## Production Scenario: The "VidiStream" Compliance Audit
**The Problem:** Your company, VidiStream, needs to maintain immutable backups of their core source code for regulatory compliance. Developers frequently push to the `main` branch, and the security team requires an off-site, versioned backup in Google Cloud Storage for every commit.
**The Task:** As the Cloud Engineer, you must deploy a "Watcher" instance that polls GitHub and archives code. However, the CTO has mandated that **no resources can be created manually**. Everything must be defined as Infrastructure as Code (IaC).

---

## Lab Objective
Architect and deploy a serverless-style automation pipeline using Terraform to manage:
1.  **Storage:** An immutable GCS Bucket.
2.  **Identity:** A least-privileged Service Account (IAM).
3.  **Compute:** A GCS-optimized VM with an automated startup sequence.

---

## Phase 1: Define the Infrastructure (The `main.tf`)
Instead of providing the code, you must construct your `main.tf` file using the following requirements. 

### 1.1 The Provider & Variables
Define a `google` provider. Use **variables** for your `project_id`, `region`, and `bucket_name` to ensure the code is reusable across different environments (Dev/Prod).

### 1.2 The Storage Layer
Create a `google_storage_bucket`. 
* **Requirement:** Enable `versioning` so that if a backup is accidentally deleted, it can be recovered.
* **Requirement:** Set `force_destroy = true` for this lab environment so you can clean up easily later.

### 1.3 The Security Layer (IAM)
In production, we **never** use the Default Compute Service Account. 
* **Task:** Create a `google_service_account` specifically for this "watcher" service.
* **Task:** Use a `google_storage_bucket_iam_member` resource to grant this account the `roles/storage.objectAdmin` role, but **only** on the bucket you created.

### 1.4 The Compute Layer
Create a `google_compute_instance`.
* **Machine Type:** Use `e2-micro` (Cost-optimization).
* **Service Account:** Attach the custom Service Account you created in 1.3.
* **Startup Script:** This is the most critical part. Use the `metadata_startup_script` attribute to automate the following:
    1.  Install `git`, `python3-pip`, and `zip`.
    2.  Clone your target GitHub repository to `/opt/repo`.
    3.  Write the `monitor.py` script (from the previous lab) to `/opt/monitor.py`.
    4.  Create the `systemd` unit file at `/etc/systemd/system/github-monitor.service`.
    5.  Reload the daemon and start the service.

---

## Phase 2: Deployment Commands
Once your code is written, you must execute the professional Terraform workflow. Open your terminal and run:

1.  **Initialize:** `terraform init` (Downloads the Google provider).
2.  **Validate:** `terraform validate` (Checks your syntax).
3.  **Plan:** `terraform plan -out=deploy.tfplan` (Review what will be built).
4.  **Apply:** `terraform apply "deploy.tfplan"` (Provision the infrastructure).

---

## Phase 3: Production Validation & Troubleshooting
Once Terraform finishes, your pipeline should be live. 
1.  **Check the VM:** SSH into the instance. Run `systemctl status github-monitor`. Is it active?
2.  **Check the Logs:** Run `journalctl -u github-monitor -f` to see the Python script's output in real-time.
3.  **Trigger the Pipeline:** Push a small change (like a README edit) to your GitHub repo.
4.  **Verify the Archive:** Check the GCS Bucket via the Console or `gsutil ls gs://YOUR_BUCKET_NAME`.

---

## PCA Architectural Challenge (Interview Prep)
To land a Cloud Architect role, you must explain *why* you built it this way. Prepare answers for the following:

* **Least Privilege:** Why did we create a custom Service Account instead of using the default one? How does this limit the "blast radius" if the VM is compromised?
* **Resiliency:** If the VM crashes and restarts, what ensures the Python script starts running again without human intervention?
* **Immutable Infrastructure:** If you need to change the GitHub URL the script monitors, should you SSH into the VM to change the script, or should you update the Terraform code and re-deploy? Why?
* **Cost Optimization:** Since this script spends most of its time "sleeping" (`time.sleep(60)`), is a 24/7 Compute Engine instance the most cost-effective choice? What serverless GCP service could replace the VM to save money?

---

### Cleanup
When finished, run:
`terraform destroy`
*This ensures you don't incur unnecessary costs—a key habit for any production-grade engineer.*