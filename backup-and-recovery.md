# 🔄 **GitLab Backup and Recovery Guide**

Ensuring the safety of your data is crucial when managing a GitLab instance. This guide explains how to back up and recover your GitLab instance using GitLab's built-in tools. Follow these steps to avoid data loss and ensure smooth recovery during unforeseen circumstances.

---

## 📋 **Table of Contents**

1. [Understanding Backup and Recovery](#understanding-backup-and-recovery)
2. [Creating a Backup](#creating-a-backup)
3. [Restoring from a Backup](#restoring-from-a-backup)
4. [Automating Backups](#automating-backups)
5. [Testing Backup and Recovery](#testing-backup-and-recovery)

---

## 1️⃣ **Understanding Backup and Recovery**

GitLab’s backup system saves important components of your instance, including:

- Repositories
- Database (PostgreSQL)
- Uploaded files (e.g., CI/CD artifacts, user avatars)
- Configuration files

The backup system generates a single `.tar` file containing these components, stored in a directory of your choice.

---

## 2️⃣ **Creating a Backup**

1. **Log in to your GitLab server** as a user with administrative privileges.

2. **Run the following command** to create a backup:
```bash
sudo gitlab-backup create
```

3. By default, backups are stored in `/var/opt/gitlab/backups/`. The filename includes the timestamp, e.g., `1674518400_gitlab_backup.tar`.

4. **Customize Backup Directory (Optional)**:  

Update the `gitlab.rb` file if you want to change the backup directory:
```bash
sudo nano /etc/gitlab/gitlab.rb
```

Add or modify the following line:
```ruby
gitlab_rails['backup_path'] = "/path/to/backup/directory"
```

Reconfigure GitLab to apply the changes:
```bash
sudo gitlab-ctl reconfigure
```

---

## 3️⃣ **Restoring from a Backup**

1. **Place the Backup File in the Backup Directory**  

Move the `.tar` backup file to the directory specified in `gitlab.rb`. By default, this is `/var/opt/gitlab/backups/`.

2. **Stop GitLab Services**  

To avoid conflicts during restoration, stop all GitLab services:
```bash
sudo gitlab-ctl stop puma
sudo gitlab-ctl stop sidekiq
```

3. **Restore the Backup**  

Run the following command, replacing `<timestamp>` with your backup file’s timestamp:
```bash
sudo gitlab-backup restore BACKUP=<timestamp>
```

Example:
```bash
sudo gitlab-backup restore BACKUP=1674518400
```

4. **Restart GitLab Services**  

Once the restoration is complete, start GitLab services:
```bash
sudo gitlab-ctl start
```

---

## 4️⃣ **Automating Backups**

1. **Schedule Backups with Cron Jobs**:  

Open the crontab editor:
```bash
sudo crontab -e
```

2. **Add a Backup Job**:  

Schedule a daily backup at midnight:
```bash
0 0 * * * /opt/gitlab/bin/gitlab-backup create
```

3. **Rotate Old Backups**:  

Enable backup retention to delete old backups automatically. Edit the `gitlab.rb` file:
```ruby
gitlab_rails['backup_keep_time'] = 604800 # 7 days in seconds
```

Reconfigure GitLab:
```bash
sudo gitlab-ctl reconfigure
```

---

## 5️⃣ **Testing Backup and Recovery**

1. **Validate Backups**:  

Periodically restore a backup on a test environment to ensure its integrity.

2. **Verify Key Components**: 

After restoration, verify the following:
- All repositories are accessible.
- User data and configurations are intact.
- CI/CD pipelines and runners work as expected.

---

## ⚠️ **Important Notes**

- **Backup Security**: Protect backup files using encryption and store them in a secure location.
- **Backup Storage**: Use external storage or cloud solutions for redundancy.
- **Downtime**: Restoring a backup requires service downtime; plan accordingly.

You should ensure the following permissions are applied to maintain security and prevent unauthorized access:

1. **Backup User Permissions**:
    
    - The user performing the backup should have **read access** to the GitLab database and repositories.
    - Ensure the user has **write access** to the backup storage location for saving the backup file.
2. **Backup Directory Permissions**:
    
    - The backup directory should have **restricted access**, granting only the GitLab backup user (or root, if necessary) **read/write** access.
    - Use `**chmod 700**` for the backup directory to ensure only the backup user can access it.
3. **Backup Storage Permissions**:
    
    - If using cloud storage or external storage, ensure that the permissions are tightly controlled:
        - Enable encryption in transit and at rest for cloud storage.
        - Limit access to specific users or roles (e.g., GitLab administrators or backup administrators).
4. **Encryption**:
    
    - Ensure that backup files are **encrypted** both in transit and at rest. This helps protect sensitive information from unauthorized access.
5. **Recovery Permissions**:
    
    - Restrict access to the recovery procedure and backup files to only trusted administrators.
    - **Ensure backups are read-only** after they are created, preventing modification.

By applying these permissions, you can better safeguard your GitLab backups and prevent unauthorized access or potential data loss.

---
