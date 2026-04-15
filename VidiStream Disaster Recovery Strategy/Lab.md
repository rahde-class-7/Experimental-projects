This lab is designed to mirror the "Scenario-Based" questions you’ll encounter on the Professional Cloud Architect (PCA) exam. It focuses on balancing operational efficiency with disaster recovery and cost-optimization.
PCA Exam Lab: "VidiStream" Disaster Recovery Strategy
1. Scenario
VidiStream, a media startup, hosts a video-processing application on a single Compute Engine instance. The application processes uploaded videos and stores them temporarily on a Zonal Persistent Disk.
The CTO has identified a single point of failure: if the zone goes down, the processed data is lost. You have been tasked with designing and implementing an automated backup system that moves data to a Multi-Regional Cloud Storage (GCS) bucket to ensure high availability and durability across regions, while minimizing "toil" (manual work).
2. Objectives
Provisioning: Deploy a Compute Engine instance with an additional non-boot Persistent Disk.
Automation: Create a Python script using Google Cloud Client Libraries to automate snapshots.
Storage Architecture: Move snapshot data to GCS and implement Lifecycle Management to control costs.
IAM & Security: Configure a Service Account with the principle of least privilege.
3. Lab Steps
Step A: Infrastructure Setup
Create a GCS Bucket: Create a bucket in a multi-region (e.g., US) to serve as your global archive.
Create a VM & Disk:
Deploy a Linux VM (e.g., e2-medium).
Create a 50GB Standard Persistent Disk and attach it to the VM as a secondary disk.
SSH into the VM, format, and mount the disk to /mnt/video-data.
Step B: Identity & Access Management (The PCA Way)
Do not use the default Compute Engine service account.
Create a custom Service Account (backup-bot).
Assign only the necessary roles:
roles/compute.storageAdmin (to create snapshots).
roles/storage.objectAdmin (to upload to GCS).
Attach this service account to your VM.
Step C: The Python Automation Script
On the VM, create backup.py. This script simulates the "automated movement" requested.
Note: In a real PCA scenario, you might use Snapshot Schedules, but for this specific lab requirement of moving data to GCS, a script or Cloud Functions is often the architectural choice for custom workflows.
from google.cloud import compute_v1
from google.cloud import storage
import datetime

# Configuration
project = "your-project-id"
zone = "us-central1-a"
disk_name = "video-data-disk"
bucket_name = "vidistream-archive-unique-id"

def create_snapshot():
    client = compute_v1.SnapshotsClient()
    snapshot_name = f"video-snap-{datetime.datetime.now().strftime('%Y%m%d-%H%M%S')}"
    
    # In a real lab, you'd trigger a disk snapshot here
    # Then export the snapshot/disk image to a tarball for GCS
    print(f"Triggering snapshot: {snapshot_name}")
    # (Simplified for lab logic: export metadata/data to GCS)
    
def upload_to_gcs(file_path, target_name):
    storage_client = storage.Client()
    bucket = storage_client.bucket(bucket_name)
    blob = bucket.blob(target_name)
    blob.upload_from_filename(file_path)
    print(f"Backup {target_name} uploaded to {bucket_name}.")

if __name__ == "__main__":
    create_snapshot()
    # Logic to upload a mock 'processed_video.mp4' from the persistent disk
    upload_to_gcs("/mnt/video-data/sample.mp4", "backups/sample_v1.mp4")


4. PCA "Pro-Tip" Knowledge Check
To pass the exam, you need to know why we chose this architecture. Match these decisions to PCA exam topics:
Decision
PCA Exam Topic
Multi-Regional GCS
Designing for 99.999999999% (11 9s) durability and regional failover.
Custom Service Account
Security & Compliance: Implementing Least Privilege.
Snapshots vs. Images
Operational Excellence: Snapshots are incremental and better for frequent backups of data disks.
GCS Lifecycle Rules
Cost Optimization: Moving older backups from "Standard" to "Coldline" or "Archive" storage classes automatically.

5. Challenge Exercise (PCA Level)
The Problem: The Python script is currently running on the VM itself. If the VM crashes, the script won't run.
The Task: How would you re-architect this to be "Serverless"?
Hint: Look into Cloud Scheduler triggering a Cloud Function.
How comfortable are you with using the gcloud CLI versus the Python SDK for these tasks?

To verify that your "VidiStream" disaster recovery lab is functioning correctly, you need to perform a Validation Test. This ensures the data flows from the Compute Engine disk all the way to Google Cloud Storage as expected.
Follow these steps to confirm the architecture is sound:
1. Data Generation (The "Input" Test)
First, ensure your Persistent Disk is actually capturing data.
Action: SSH into your VM and create a dummy video file on the mounted disk.
Command: ```bashCreate a 10MB dummy file to simulate a processed videodd if=/dev/urandom of=/mnt/video-data/test_video_01.mp4 bs=1M count=10


Verification: Run ls /mnt/video-data to confirm test_video_01.mp4 exists.
2. Execution & IAM Validation
Run your Python script to see if it can communicate with the GCP APIs using the Service Account you attached.
Action: Run your script: python3 backup.py.
Success Indicators:
No "403 Forbidden" errors (This proves your IAM Roles are correct).
Console output confirms "Snapshot triggered" and "Upload successful."
3. The "Single Source of Truth" Check (Cloud Storage)
The ultimate proof of success for the PCA exam is seeing the data in the bucket.
Action: Use the gcloud CLI or the Console to check your bucket.
Command: ```bash gsutil ls gs://[YOUR_BUCKET_NAME]/backups/


Expected Result: You should see sample_v1.mp4 (or whatever naming convention you used in your script) listed in the bucket.
4. Lifecycle Policy Simulation (Cost Check)
Since PCA focuses on Cost Optimization, you should verify that your bucket will automatically move these files to cheaper storage over time.
Action: Check the lifecycle configuration of your bucket.
Command:
gsutil lifecycle get gs://[YOUR_BUCKET_NAME]


Verification: Ensure the output shows a rule where action: SetStorageClass is COLDLINE or ARCHIVE after a certain number of days (e.g., 30 days).
5. The "Disaster" Recovery Test
This is the most important test for a Cloud Architect.
Simulate Failure: Delete the secondary Persistent Disk (video-data-disk) from the GCP Console.
The Recovery: * Create a new Persistent Disk.
Use the gsutil cp command to pull your "backup" from the GCS bucket back onto the new disk.
Verify: Run md5sum on the original file and the recovered file to ensure no data corruption occurred during the move.
Summary Checklist for Success
Component
Success Criteria
Compute Engine
Can write files to /mnt/video-data.
IAM
VM can call GCS and Compute APIs without using a password/key file.
Python Script
Successfully automates the upload method.
Cloud Storage
Files appear in the bucket with the correct storage class.

Would you like me to show you how to set up the GCS Lifecycle rules using the gcloud command to round out the automation?
