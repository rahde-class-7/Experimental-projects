This lab focuses on building a Serverless-style Automation Pipeline using a persistent Compute Engine instance. You will use Python as the orchestration engine to "poll" GitHub for changes and archive those changes into an immutable Google Cloud Storage (GCS) bucket.

## Lab Title: Automated Source Archiving with Python and GCS
### Objective
To configure a GCP VM that monitors a specific Git branch and automatically backs up the repository to a Cloud Storage bucket whenever a new commit is detected.

### Phase 1: Cloud Storage Setup
Before the VM can store data, you need a destination.
In the GCP Console, go to Cloud Storage > Buckets.
Click Create.
Name: Give it a globally unique name (e.g., archive-repo-project-99).
Location: Choose the same region as your VM to minimize data transfer costs.
Keep all other defaults and click Create.
### Phase 2: Granting Permissions (IAM)
For the VM to write files to the bucket, it needs a "identity."
Go to IAM & Admin > Service Accounts.
Find the service account used by your VM (usually the "Compute Engine default service account").
Click Edit (pencil icon) and click Add Another Role.
Select Storage Object Admin. This allows the VM to upload files.
Click Save.

### Phase 3: The Python Monitor Script
You will now write a script that acts as the "bridge" between GitHub and GCS.
SSH into your VM.
Install the Google Cloud Storage library:
pip3 install google-cloud-storage


Create a file named monitor.py:
within the file paste the enite code

```
import os
import time
import subprocess
from google.cloud import storage

# Configuration
REPO_DIR = "/home/your_user/my_repo"
BUCKET_NAME = "your-unique-bucket-name"
CHECK_INTERVAL = 60 # Seconds

client = storage.Client()
bucket = client.bucket(BUCKET_NAME)

def get_latest_commit():
    os.chdir(REPO_DIR)
    subprocess.run(["git", "pull"], capture_output=True)
    return subprocess.check_output(["git", "rev-parse", "HEAD"]).decode().strip()

last_commit = ""

while True:
    current_commit = get_latest_commit()
    if current_commit != last_commit:
        print(f"New change detected: {current_commit}. Archiving...")

        # Zip the repo and upload
        zip_name = f"backup_{current_commit}.zip"
        subprocess.run(["zip", "-r", zip_name, "."])

        blob = bucket.blob(zip_name)
        blob.upload_from_filename(zip_name)

        print(f"Successfully uploaded {zip_name} to GCS.")
        last_commit = current_commit
        os.remove(zip_name)

    time.sleep(CHECK_INTERVAL)
```

### Phase 4: Scheduling and Execution
In production, you don't want to run this manually. You will use a Systemd Service to ensure the script starts automatically if the VM reboots.
Create a service file:
sudo nano /etc/systemd/system/github-monitor.service

```
Paste the following configuration:
[Unit]
Description=GitHub to GCS Sync Service
After=network.target

[Service]
User=your_username
WorkingDirectory=/home/your_username
ExecStart=/usr/bin/python3 /home/your_username/monitor.py
Restart=always

[Install]
WantedBy=multi-user.target
```

Start the service:
```
sudo systemctl enable github-monitor
sudo systemctl start github-monitor
```

### Phase 5: Validation
Push a new commit to your GitHub repository from your local computer.
Wait 60 seconds.
Go to the Google Cloud Storage console and refresh your bucket.
You should see a new .zip file named after the Git commit hash.
PCA Architectural Reflection
Decoupling: Why is it better to store archives in Cloud Storage rather than keeping them on the VM’s Persistent Disk?
Reliability: By using a systemd service with Restart=always, how are you improving the Availability of your monitoring solution?
Security: How does using a Service Account instead of hardcoding an API Key into the Python script improve the security posture of the lab?
