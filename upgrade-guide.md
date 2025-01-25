# ⬆️ **GitLab Upgrade Guide**

Keeping GitLab up to date is essential to ensure access to the latest features, bug fixes, and security patches. This guide provides step-by-step instructions for upgrading your GitLab instance, whether you're using the Omnibus package, Docker, or source installation.

---

## 📋 **Table of Contents**

1. [Preparing for an Upgrade](#preparing-for-an-upgrade)
2. [Upgrading Using Omnibus](#upgrading-using-omnibus)
3. [Upgrading Using Docker](#upgrading-using-docker)
4. [Upgrading a Source Installation](#upgrading-a-source-installation)
5. [Post-Upgrade Steps](#post-upgrade-steps)

---

## 1️⃣ **Preparing for an Upgrade**

1. **Review the Release Notes**  

Check the [GitLab Release Notes](https://docs.gitlab.com/ee/releases/) to understand the changes in the new version.

2. **Backup Your GitLab Instance**  

Always back up your GitLab instance before upgrading to prevent data loss in case of any issues:
```bash
sudo gitlab-backup create
```

3. **Check Compatibility**  

Verify that your current version supports a direct upgrade to the target version. If not, upgrade incrementally as per the [upgrade paths](https://docs.gitlab.com/ee/update/#upgrade-paths).

4. **Schedule Downtime**  

Upgrades may require service downtime, so plan accordingly. Notify your team about the maintenance window.

---

## 2️⃣ **Upgrading Using Omnibus**

1. **Update the GitLab Package Repository**  

Update your package manager’s repository to fetch the latest GitLab version:
```bash
sudo apt-get update
```

2. **Upgrade GitLab**  

Run the following command to upgrade:
```bash
sudo apt-get install gitlab-ee
```

- Replace `gitlab-ee` with `gitlab-ce` if you're using the Community Edition.

3. **Reconfigure GitLab**  

After the upgrade, reconfigure GitLab to apply changes:
```bash
sudo gitlab-ctl reconfigure
```

4. **Restart Services**  

Restart all GitLab services to ensure they’re running correctly:
```bash
sudo gitlab-ctl restart
```

---

## 3️⃣ **Upgrading Using Docker**

1. **Pull the Latest GitLab Image**  

Use the `docker pull` command to download the latest GitLab Docker image:
```bash
docker pull gitlab/gitlab-ee:latest
```

For the Community Edition, use `gitlab/gitlab-ce:latest`.

2. **Stop the Existing Container**  

Stop the running GitLab container:
```bash
docker stop <container-name>
```

3. **Remove the Old Container**  

Remove the old container (the data will remain intact as it's stored in volumes):
```bash
docker rm <container-name>
```

4. **Start the New Container**  

Run the new container with the latest image:
```bash
docker run --detach \
  --hostname gitlab.example.com \
  --name gitlab \
  --restart always \
  --volume /srv/gitlab/config:/etc/gitlab \
  --volume /srv/gitlab/logs:/var/log/gitlab \
  --volume /srv/gitlab/data:/var/opt/gitlab \
  gitlab/gitlab-ee:latest
```

Adjust volume paths as per your setup.

---

## 4️⃣ **Upgrading a Source Installation**

1. **Backup Your Database**  

Ensure the PostgreSQL database is backed up using `pg_dump` or another method.

2. **Clone the Latest Version**  

Navigate to your GitLab directory and pull the latest stable release:
```bash
git fetch --all
git checkout <new-version>
```

3. **Update Dependencies**  

Install any new dependencies required for the update:
```bash
bundle install
yarn install
```

4. **Migrate the Database**  

Run database migrations to apply any schema changes:
```bash
bundle exec rake db:migrate RAILS_ENV=production
```

5. **Compile Assets**  

Precompile the assets for the new version:
```bash
bundle exec rake assets:precompile RAILS_ENV=production
```

6. **Restart GitLab**  

Restart GitLab to apply the updates:
```bash
sudo systemctl restart gitlab
```

---

## 5️⃣ **Post-Upgrade Steps**

1. **Verify the Upgrade**  

Check the GitLab version to confirm the upgrade:
```bash
gitlab-rake gitlab:env:info
```

2. **Test Core Features**  

Verify that repositories, pipelines, and other essential features are functioning correctly.

3. **Check Logs for Errors**  

Review the logs for any issues:
```bash
sudo gitlab-ctl tail
```

4. **Update Runners (Optional)**  

If there are updates to GitLab Runners, ensure they are also upgraded for compatibility.

---

## ⚠️ **Important Notes**

- Always test upgrades in a staging environment before applying them to production.
- Avoid skipping major version upgrades unless explicitly supported.
- If you encounter issues during the upgrade, consult the [GitLab Upgrade Troubleshooting Guide](https://docs.gitlab.com/ee/update/#troubleshooting)
