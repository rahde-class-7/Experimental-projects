This GitHub README provides a professional structure for your project, including architectural insights that align with your focus on Google Cloud.
# Automated Source Archiving with Python and GCS
This repository contains a **Serverless-style Automation Pipeline** that leverages a Compute Engine instance to provide continuous data redundancy. Using Python as an orchestration engine, the system polls a GitHub repository for changes and archives them into an immutable Google Cloud Storage (GCS) bucket upon every new commit.
## 🏗️ Architecture Overview
The pipeline operates by decoupling the source control from the archival storage, ensuring that even if the compute instance or the original repository is compromised, a versioned history remains secure in GCS.
 * **Compute Engine:** Hosts the Python orchestration script and performs the polling logic.
 * **Python (monitor.py):** Periodically checks for new Git commit hashes using subprocess and handles the compression and upload logic.
 * **Cloud Storage:** Acts as the immutable destination for repository backups.
 * **Systemd:** Ensures the monitoring service remains highly available and resilient to reboots.
## 🚀 Lab Phases
### Phase 1: Cloud Storage Setup
 1. Create a **globally unique** bucket in the GCP Console.
 2. Select a region that matches your VM to minimize egress costs and latency.
### Phase 2: IAM & Permissions
This project follows the **Principle of Least Privilege**.
 * **Identity:** Compute Engine Default Service Account.
 * **Role:** Storage Object Admin.
 * **Security Note:** No hardcoded API keys; the script utilizes Application Default Credentials (ADC) via the VM's identity.
### Phase 3: The Python Monitor Script
The core logic resides in monitor.py. It compares the current HEAD commit against the previously recorded hash.
```python
# Key functionality:
# 1. Pulls latest changes from Git
# 2. Compares commit hashes
# 3. Zips the directory and uploads via google-cloud-storage library

```
### Phase 4: Persistence with Systemd
To ensure the script runs as a background daemon and survives system restarts, a Systemd service is implemented:
```bash
sudo systemctl enable github-monitor
sudo systemctl start github-monitor

```
## 🧠 PCA Architectural Reflection
### 1. Decoupling
**Why store archives in GCS rather than Persistent Disk?**
Storing archives in GCS separates the storage lifecycle from the compute lifecycle. Persistent Disks are tied to a specific zone and instance; GCS provides **99.999999999% (11 9's)** durability and makes the data accessible globally without the need for an active VM.
### 2. Reliability & Availability
**How does the Systemd service improve the solution?**
By configuring Restart=always, the solution becomes self-healing. If the Python script crashes due to a network timeout or memory leak, Systemd will automatically restart the process, maintaining the "Availability" pillar of the Well-Architected Framework.
### 3. Security Posture
**Why use Service Accounts over API Keys?**
API keys are long-lived and easily leaked if committed to version control. Service Accounts use short-lived, automatically rotated tokens managed by Google. This reduces the blast radius and eliminates the need for manual secret management within the source code.
## 🛠️ Requirements
 * Google Cloud Platform Account
 * Compute Engine (Linux)
 * Python 3.x
 * google-cloud-storage Python library
 * git and zip utilities installed on the VM
