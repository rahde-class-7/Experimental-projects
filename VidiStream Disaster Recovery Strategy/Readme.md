This README is structured to highlight the architectural principles emphasized in the Google Cloud Professional Cloud Architect (PCA) exam, focusing on resilience, security, and cost-optimization.
# VidiStream: Automated Disaster Recovery Pipeline
## 📖 Project Overview
VidiStream is a media processing solution designed to eliminate single points of failure. This project implements an automated backup pipeline that migrates data from **Zonal Persistent Disks** to **Multi-Regional Cloud Storage**, ensuring high availability and 11 9's of durability.
## 🏗️ Architecture
The solution follows the Google Cloud Well-Architected Framework by decoupling compute from stateful storage.
 * **Compute:** Engine instance (e2-medium) processing video data.
 * **Primary Storage:** 50GB Standard Persistent Disk mounted at /mnt/video-data.
 * **Archive Storage:** Multi-Regional GCS Bucket for regional failover capability.
 * **Orchestration:** Python-based automation using Google Cloud Client Libraries.
 * **Security:** Custom Service Account (backup-bot) leveraging the Principle of Least Privilege.
## 🛠️ Implementation Steps
### 1. Infrastructure Provisioning
 * Deploy a Linux VM and attach a secondary 50GB Persistent Disk.
 * Format and mount the disk:
   ```bash
   sudo mkfs.ext4 -m 0 -E lazy_itable_init=0,lazy_journal_init=0,discard /dev/sdb
   sudo mkdir -p /mnt/video-data
   sudo mount -o discard,defaults /dev/sdb /mnt/video-data
   
   ```
### 2. IAM Configuration (The "PCA Way")
Avoid using the Default Compute Service Account. Instead:
 1. Create backup-bot service account.
 2. Bind roles/compute.storageAdmin for snapshot management.
 3. Bind roles/storage.objectAdmin for GCS uploads.
### 3. Automation Logic
The backup.py script orchestrates the lifecycle of a backup by triggering snapshots and exporting processed assets to the global archive.
```python
# key logic snippet
def upload_to_gcs(file_path, target_name):
    storage_client = storage.Client()
    bucket = storage_client.bucket(bucket_name)
    blob = bucket.blob(target_name)
    blob.upload_from_filename(file_path)

```
## 🎓 PCA Architectural Mapping
| Decision | Exam Domain | Rationale |
|---|---|---|
| **Multi-Regional GCS** | Availability | Ensures data survives even if an entire GCP region faces an outage. |
| **Custom Service Account** | Security | Minimizes "blast radius" by granting only necessary permissions. |
| **Snapshots** | Operations | Leverages incremental backup technology to reduce storage costs and time. |
| **Lifecycle Rules** | Cost Optimization | Automatically transitions old backups to Coldline/Archive classes. |
## 🚀 Advanced Challenge: Going Serverless
To improve the **Reliability** pillar, the current VM-based script should be re-architected to remove the dependency on a running instance:
 1. **Cloud Scheduler:** Replaces the manual/cron trigger.
 2. **Pub/Sub:** Acts as the message bus between the scheduler and the compute.
 3. **Cloud Functions:** Executes the Python logic in a serverless environment, triggering the snapshot and GCS transfer without managing underlying servers.
## 🛠️ Technical Stack
 * **Language:** Python 3.x
 * **Libraries:** google-cloud-compute, google-cloud-storage
 * **CLI:** gcloud SDK
Regarding your comfort level with the tools, the Python SDK is excellent for building the logic *within* an application, but mastering the gcloud CLI is often faster for infrastructure "plumbing" and is a major component of the PCA exam's practical logic.
Would you like to see how the gcloud equivalent commands for Step A and B compare to the Python implementation?
