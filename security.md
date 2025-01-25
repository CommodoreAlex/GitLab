# 🔒 **GitLab Security Guide**

Security is paramount when running a GitLab instance, whether for personal or team use. This guide will cover the necessary steps and best practices to secure your GitLab instance, including configuring HTTPS, user access control, setting up two-factor authentication (2FA), and monitoring system activity to detect potential threats.

**Note: Some of this is a refresher on previously covered material in other sections, compounding the value of security best practices and technical processes.**

---

## 📋 **Table of Contents**

1. [Enabling HTTPS](#enabling-https)
2. [User Access Control](#user-access-control)
3. [Two-Factor Authentication (2FA)](#two-factor-authentication-2fa)
4. [Monitoring and Logging](#monitoring-and-logging)
5. [Regular Updates and Patches](#regular-updates-and-patches)
6. [Backup and Encryption](#backup-and-encryption)
7. [General Security Best Practices](#general-security-best-practices)

---

## 1️⃣ **Enabling HTTPS**

**Why HTTPS?**  
Enabling HTTPS ensures that all communication between users and your GitLab instance is encrypted, preventing unauthorized access to sensitive data.

**Steps to Enable HTTPS:**

1. **Obtain an SSL/TLS certificate**: You can use a free service like [Let’s Encrypt](https://letsencrypt.org/) or a paid certificate from a trusted certificate authority (CA).

2. **Configure SSL in GitLab**:  

Open the `gitlab.rb` file for editing:
 ```bash
sudo nano /etc/gitlab/gitlab.rb
```

3. Add or update the `external_url` and SSL settings:

```ruby
external_url 'https://yourdomain.com'
nginx['listen_https'] = true
nginx['ssl_certificate'] = "/etc/ssl/certs/gitlab.crt"
nginx['ssl_certificate_key'] = "/etc/ssl/private/gitlab.key"
```

4. **Reconfigure GitLab** to apply changes:

```bash
sudo gitlab-ctl reconfigure
```

---

## 2️⃣ **User Access Control**

**Best Practices:**  
Restricting access to your GitLab instance is essential to protect your projects and data. Follow these best practices for managing user access:

1. **Admin User Setup:**  
    Set up strong authentication for the root admin user. Avoid using "root" for everyday operations. Create a separate admin user and assign appropriate permissions.
    
2. **Role-Based Access Control (RBAC):**  
    Use GitLab’s built-in roles (Guest, Reporter, Developer, Maintainer, and Owner) to control access to repositories and features based on user roles.
    
3. **Restrict SSH Key Usage:**  
    Require SSH key authentication for Git operations and restrict password-based access. Enforce SSH key usage for all Git operations.
    
4. **IP Whitelisting:**  
    Restrict access to your GitLab instance by allowing only certain IPs to access the server, especially for sensitive operations like administrative tasks.
    

---

## 3️⃣ **Two-Factor Authentication (2FA)**

**Why 2FA?**  
Two-factor authentication adds an extra layer of security by requiring both a password and a second factor (such as a phone app) to authenticate.

**Steps to Enable 2FA:**

1. **Enable 2FA for All Users:**

Edit the `gitlab.rb` file:
```bash
sudo nano /etc/gitlab/gitlab.rb
```

Set the following value to enforce 2FA:
```ruby
gitlab_rails['gitlab_shell_ssh_port'] = 22
gitlab_rails['gitlab_2fa'] = true
```

2. **Reconfigure GitLab** to apply the changes:

```bash
sudo gitlab-ctl reconfigure
```

3. **User Setup:**  

Users can enable 2FA from their GitLab profile under **User Settings > Account > Two-Factor Authentication**.

---

## 4️⃣ **Monitoring and Logging**

**Why Monitoring?**  
Continuous monitoring and logging help detect unauthorized access, unusual behavior, and potential security threats in real-time.

**Steps to Set Up Monitoring and Logging:**

1. **Enable GitLab Logging**:  

GitLab logs various actions, including login attempts and server events. To monitor these logs, use the following command to view real-time logs:
```bash
sudo gitlab-ctl tail
```


2. **Set Up External Monitoring Tools:**  

Integrate GitLab with monitoring tools like [Prometheus](https://prometheus.io/) or [Grafana](https://grafana.com/), which can provide real-time monitoring of GitLab's performance and security metrics.

3. **Enable Audit Logging (for Enterprise Editions):**  

If you have GitLab EE (Enterprise Edition), enable Audit Logging to track and record user activity for compliance purposes. Audit logs can be found under **Admin Area > Logs > Audit Events**.

4. **Configure Alerts:**  

Set up alerts for suspicious activities (e.g., multiple failed login attempts, admin changes) using GitLab’s internal alerting system or third-party tools.

---

## 5️⃣ **Regular Updates and Patches**

**Why Updates?**  
Regular updates ensure that your GitLab instance is protected against known vulnerabilities and exploits.

**Steps for Regular Updates:**

1. **Update GitLab:**

Run the following command to update your GitLab instance to the latest stable version:
```bash
sudo apt-get update
sudo apt-get install gitlab-ce
```


2. **Check for Updates:**  

To check for the latest GitLab updates, visit the official [GitLab Release Page](https://about.gitlab.com/releases/).

3. **Apply Security Patches:**  

Ensure you’re applying security patches as soon as they are released. GitLab provides security advisories to keep track of important security fixes.

4. **Reconfigure After Updates:**  

After updates or changes, always reconfigure GitLab to ensure all services are running properly:
```bash
sudo gitlab-ctl reconfigure
```

---

## 6️⃣ **Backup and Encryption**

**Why Backups and Encryption?**  
Encrypting sensitive data and having reliable backups are crucial to protecting your GitLab instance and ensuring business continuity in case of data loss or breaches.

**Steps for Backup and Encryption:**

1. **Backup GitLab Data:**  

Create backups of your GitLab instance using the following command:
```bash
sudo gitlab-backup create
```

2. **Encrypt Backup Files:**  

To add an extra layer of protection, you can encrypt your backup files:
```bash
sudo gpg -c /var/opt/gitlab/backups/your-backup.tar
```

3. **Store Backups Securely:**  

Ensure your backups are stored in a secure location, ideally offsite or in a cloud storage service that is encrypted.

4. **Encrypt Sensitive Data at Rest:**  

Enable encryption for sensitive data stored in GitLab, such as passwords and other confidential information, by configuring the `gitlab.rb` file:
```ruby
gitlab_rails['gitlab_shell_ssh_port'] = 22
gitlab_rails['password_authentication'] = false
```

---

## ⚠️ **General Security Best Practices:**

- **Use Strong Passwords:** Ensure that all users follow the principle of using strong, unique passwords.
- **Limit Admin Access:** Minimize the number of users with admin access and assign them only when necessary.
- **Regularly Audit User Accounts:** Periodically review user access and disable any unused or suspicious accounts.
- **Encrypt Communications:** Use HTTPS for secure communications and SSH for Git operations.
- **Monitor Security Bulletins:** Stay informed by subscribing to GitLab’s security mailing list for updates on vulnerabilities and fixes.
- **Avoid Hard-Coding Credentials:** One of the most common ways attackers gain access to internal/external resources on servers and networks is through the discovery of hard-coded credentials in code repositories or configuration files.
- **Monitor for Public Directory Scanning:** Public repositories can be scanned by tools that search for sensitive data like hard-coded credentials. Ensure you audit all public-facing GitLab directories and prevent accidental exposure of sensitive information.

---

By following these security practices, you’ll be able to maintain a secure GitLab instance and protect your repositories and data from unauthorized access. For more information, refer to the official [GitLab Security Documentation](https://docs.gitlab.com/ee/administration/security/).
