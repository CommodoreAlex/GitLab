
# Configuration Guide

Once you have installed GitLab, it's essential to configure it to meet your requirements. This guide walks you through the key configuration steps to set up your GitLab instance for production or personal use.

---

## Table of Contents

1. [External URL Configuration](#external-url-configuration)
2. [Initial Login and Admin Setup](#initial-login-and-admin-setup)
4. [Setting Up HTTPS](#setting-up-https)
5. [Adding GitLab Runners](#adding-gitlab-runners)
6. [Email Notifications](#email-notifications)
7. [Advanced Configurations](#advanced-configurations)

---

## External URL Configuration

1. **Edit the GitLab Configuration File

```bash
sudo nano /etc/gitlab/gitlab.rb
```

2. **Set the External URL**

Locate the line starting with `external_url` and update it with your domain or IP (or localhost):
```ruby
external_url 'http://your-domain.com'
```

3. **Reconfigure GitLab**

Apply the changes:
```bash
sudo gitlab-ctl reconfigure
```

---
## Reconfiguring takes time

You need to wait for GitLab to finish setting up. The reconfiguration process can take a little time as it sets up all the services and dependencies necessary for GitLab to function properly.

Here’s what typically happens during the setup:

- GitLab configures internal services (e.g., PostgreSQL, Redis, NGINX).
- It sets up the necessary directories and files.
- It checks for any required dependencies or missing configurations.

Once the reconfiguration is complete, you'll see a message indicating that GitLab is ready to be accessed.

You can check if the process is still running by running:
```bash
sudo gitlab-ctl status
```

This command will show you the status of all GitLab services. When they’re all "running," you're good to go, and you can then access GitLab in your browser.

This is what you could expect to see:
```bash
┌──(root㉿kali)-[/home/kali]
└─# gitlab-ctl status
run: gitaly: (pid 34505) 156s; run: log: (pid 34558) 153s
run: gitlab-kas: (pid 34926) 134s; run: log: (pid 34967) 131s
run: logrotate: (pid 34354) 168s; run: log: (pid 34378) 167s
run: postgresql: (pid 34660) 145s; run: log: (pid 34701) 142s
run: redis: (pid 34435) 162s; run: log: (pid 34460) 161s
```

`gitaly`, `gitlab-kas`, `logrotate`, `postgresql`, and `redis` are all active, with their respective processes running.

This means GitLab is up and running. Now, you should be able to access it through your browser by visiting `http://localhost` or `http://127.0.0.1` (if you're configured in lab, otherwise use your configuration parameters you set).

This is what you should see on port 80:
![2](https://github.com/user-attachments/assets/aa1ce7cb-8ff7-4b78-96fd-5114050b1cbe)

If you still can’t access it, make sure that any firewall rules aren’t blocking the ports GitLab uses (like port 80 or 443), or check if the server is properly configured to allow web traffic on those ports.

---
## Initial Login and Admin Setup

1. **Access GitLab**

Open your browser and navigate to the external URL or IP address where GitLab is installed.

2. **Login as Admin**

![3](https://github.com/user-attachments/assets/9d370312-eb6f-4bb1-b5fa-0a7f42738ae1)

Use the default admin credentials:
* ***Username:** `root`
* ***Password:** Check the logs during installation or use the reset method below.

3. **Resetting the Admin Password (if needed)**

Run the following command on your server to set a new password (documentation: https://docs.gitlab.com/ee/security/reset_user_password.html#reset-your-root-password):
```bash
sudo gitlab-rails console
user = User.find_by_username('root')
user.password = 'NewPassword'
user.password_confirmation = 'NewPassword'
user.save!
exit
```

**You can also find your password locally by visiting the initial configuration file at `/etc/gitlab/initial_root_password`:**
![1](https://github.com/user-attachments/assets/bf764ff2-f8b6-40b7-8e7a-6ccb4ac69879)

4. **Create Additional Users**

Navigate to the **Admin Area** > **Users** to create or manage accounts.

---

## Setting Up HTTPS

Securing your GitLab instance with HTTPS is critical for production environments.

1. **Obtain an SSL Certificate

Use a tool like [Let's Encrypt](https://letsencrypt.org/):
```bash
sudo apt install certbot
sudo certbot certonly --standalone -d your-domain.com
```

2. **Update the Configuration File*

Edit the `gitlab.rb` file:
```bash
sudo nano /etc/gitlab/gitlab.rb
```

Set the paths to your SSL certificate and key:
```ruby
external_url 'https://your-domain.com'
nginx['ssl_certificate'] = "/etc/letsencrypt/live/your-domain.com/fullchain.pem"
nginx['ssl_certificate_key'] = "/etc/letsencrypt/live/your-domain.com/privkey.pem"
     ```

3. **Reconfigure GitLab**

```bash
sudo gitlab-ctl reconfigure
```

## Setting up HTTPS with Localhost

You cannot obtain a valid TLS/SSL certificate from Let's Encrypt for `localhost` because Let's Encrypt requires domain names with at least one dot, such as `example.com` or `gitlab.local`.

If you're working with `127.0.0.1` or `localhost`, you have a couple of options:

---

### 1️⃣ Use a Self-Signed Certificate (Recommended for Local Development)

You can generate a self-signed certificate for `localhost`. Here's how:

1. **Generate the Self-Signed Certificate:**
```bash
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout localhost.key -out localhost.crt -subj "/CN=localhost"
```

This creates two files:

- `localhost.key`: The private key.
- `localhost.crt`: The self-signed certificate.

2.  **Place the Certificate in the Proper Directory:** Move the files to a location where your web server can use them, e.g., `/etc/gitlab/ssl/`:
```bash
sudo mkdir -p /etc/gitlab/ssl
sudo mv localhost.key /etc/gitlab/ssl/
sudo mv localhost.crt /etc/gitlab/ssl/
```

3. **Configure GitLab to Use the Certificate:** Edit `/etc/gitlab/gitlab.rb` and specify the SSL certificate paths:

```ruby
external_url "https://127.0.0.1"
nginx['ssl_certificate'] = "/etc/gitlab/ssl/localhost.crt"
nginx['ssl_certificate_key'] = "/etc/gitlab/ssl/localhost.key"
```

4. **Reconfigure GitLab:** Apply the changes:
```bash
sudo gitlab-ctl reconfigure
```

---

### 2️⃣ Use a Custom Domain with `/etc/hosts`

If you want a Let's Encrypt certificate, you must use a domain name. You can create a fake domain locally by modifying `/etc/hosts`:

1. **Edit `/etc/hosts`:** Add a custom domain pointing to `127.0.0.1`:
```bash
sudo nano /etc/hosts
```

Add a line:
```bash
127.0.0.1   gitlab.local
```

2. **Obtain a Certificate for `gitlab.local`:** Run Certbot using the fake domain:

```bash
certbot certonly --standalone -d gitlab.local --register-unsafely-without-email
```

3. **Configure GitLab to Use the Certificate:** Place the certificate files in `/etc/gitlab/ssl/` and update the `gitlab.rb` file as described above.

---
### 3️⃣ Skip HTTPS for Local Development

If security isn't critical (e.g., for local development/testing only), you can skip HTTPS:

1. **Update `gitlab.rb`:** Set the `external_url` to use HTTP instead of HTTPS:
```ruby
external_url "http://127.0.0.1"
```

2. **Reconfigure GitLab:**
```bash
sudo gitlab-ctl reconfigure
```

---

## Adding GitLab Runners

GitLab Runner is a lightweight application used to execute **CI/CD pipelines** (Continuous Integration and Continuous Deployment) for projects hosted in GitLab. It runs jobs defined in your `.gitlab-ci.yml` file and communicates with your GitLab instance to receive and execute instructions.

1. **Install GitLab Runner**

Follow the [official GitLab Runner installation guide](https://docs.gitlab.com/runner/install/).

Installing GitLab Runner (there are workarounds for Debian):
```bash
curl -L https://packages.gitlab.com/install/repositories/runner/gitlab-runner/script.deb.sh | sudo bash
sudo apt-get install gitlab-runner
```

2. **Register the Runner**

Use the token from **Settings > CI/CD > Runners**:
```bash
sudo gitlab-runner register
```

Follow the prompts to configure the runner:
* URL: `http://your-domain.com`
* Token: Paste the registration token.
* Executor: Choose `docker`, `shell`, etc.

Without going too far into it: Example `.gitlab-ci.yml`

Here’s a simple example of how a GitLab Runner is used in a pipeline:
```yaml
stages:
  - build
  - test

build-job:
  stage: build
  script:
    - echo "Building the app..."
    - make build

test-job:
  stage: test
  script:
    - echo "Running tests..."
    - make test
```

When this file is committed, GitLab will send these jobs to a GitLab Runner for execution. A pipeline typically includes steps for installing dependencies, running tests, building the application, and deploying it.

---

## Email Notifications

1. **Set Up SMTP

Edit the `gitlab.rb` file:
```bash
sudo nano /etc/gitlab/gitlab.rb
```

Add the SMTP settings:
```ruby
gitlab_rails['smtp_enable'] = true
gitlab_rails['smtp_address'] = "smtp.example.com"
gitlab_rails['smtp_port'] = 587
gitlab_rails['smtp_user_name'] = "your-email@example.com"
gitlab_rails['smtp_password'] = "your-password"
gitlab_rails['smtp_domain'] = "example.com"
gitlab_rails['smtp_authentication'] = "login"
gitlab_rails['smtp_enable_starttls_auto'] = true
```

2. **Reconfigure GitLab**

```bash
sudo gitlab-ctl reconfigure
```

---

 **Email Notifications for Localhost**

1. **Install and Configure Postfix**

First, install Postfix, a mail transfer agent (MTA) that can act as an SMTP server for localhost.
```bash
sudo apt update && sudo apt install postfix -y
```

During installation, choose **"Local only"** when prompted by Postfix's configuration.

When configuring a mail system like Postfix for **localhost**, the **system mail name** is essentially the domain name that your system will use to identify itself when sending emails. For a local environment, this doesn't need to be a real domain name—it can simply be `localhost` or something descriptive like `my-localhost`.

Suggested Mail Name Options for Localhost:
1. **`localhost`**: The simplest and most common option for local setups.
2. **`your-machine-name.local`**: If you want it tied to your system's hostname, you can use something like `my-computer.local`.
3. **Custom Name**: You can use something generic like `dev.local` or `local.mail`.

Alternatively, if you're manually editing Postfix configuration, open the `/etc/postfix/main.cf` file:
```bash
sudo nano /etc/postfix/main.cf
```

Look for the `myhostname` and `myorigin` settings and set them like this:
```bash
myhostname = localhost
myorigin = $myhostname
```

After Configuration restart Postfix to apply the changes:
```bash
sudo systemctl restart postfix
```

You can test your local setup by sending a test email using the `mail` command (install with `sudo apt install mailutils` if needed):
```bash
echo "This is a test email" | mail -s "Test Email" your-email@example.com
```

---

2. **Edit the GitLab SMTP Configuration**

Edit the `gitlab.rb` file:
```bash
sudo nano /etc/gitlab/gitlab.rb
```

Set up the SMTP settings to use your localhost:
```ruby
gitlab_rails['smtp_enable'] = true
gitlab_rails['smtp_address'] = "127.0.0.1"  # Localhost IP
gitlab_rails['smtp_port'] = 25              # Default SMTP port
gitlab_rails['smtp_domain'] = "localhost"
gitlab_rails['smtp_authentication'] = "none"
gitlab_rails['smtp_enable_starttls_auto'] = false
gitlab_rails['smtp_tls'] = false
```

3. **Reconfigure GitLab**

Apply the configuration changes:
```bash
sudo gitlab-ctl reconfigure
```

4. **Test Email Notifications**

To verify that email notifications are working, send a test email:
```bash
sudo gitlab-rails console
```

In the console, run:
```ruby
Notify.test_email('your-email@example.com', 'Test Subject', 'This is a test email.').deliver_now
```

If the configuration is correct, you should receive an email at the specified address.

### **Additional:**

- **Security**: This setup is for local development and testing purposes. For production, always use a secure SMTP server with proper authentication.
- **Firewall**: Ensure port `25` is open on your system if you expect to access the SMTP server from other hosts.
- **Mail Delivery**: If you're not receiving the email, check Postfix logs for errors using:
```bash
sudo tail -f /var/log/mail.log
```

---

## Advanced Configurations

1. **Optimize Performance**

Adjust the number of Unicorn workers based on your CPU:
```bash
sudo nano /etc/gitlab/gitlab.rb
```

Changes you're making:
```ruby
unicorn['worker_processes'] = 4
```

Reconfigure GitLab:
```bash
sudo gitlab-ctl reconfigure
```

### **Setting Up MFA in GitLab**

#### 1. **Enable MFA for Your Account**

1. Log in to your GitLab account.
2. Go to your profile settings:
    - Click on your profile picture in the top-right corner and select **"Edit profile."**
3. Navigate to **"Account"** > **"Two-factor Authentication."**

#### 2. **Set Up an Authenticator App**

1. Install a Time-Based One-Time Password (TOTP) authenticator app on your phone, such as:
* [Microsoft Authenticator](https://www.microsoft.com/en-us/security/mobile-authenticator-app)

3. On the **Two-factor Authentication** page in GitLab, click **"Register with your device."**

4. Scan the QR code displayed on the page using your authenticator app.
* If you can’t scan the QR code, GitLab provides a manual code you can enter into the app.

5. Your app will generate a 6-digit code. Enter this code in the **verification field** on GitLab and click **"Enable two-factor authentication."**

#### 3. **Save Your Recovery Codes**

- Once MFA is enabled, GitLab will generate a set of recovery codes. These are essential in case you lose access to your authenticator app.
- Download or print the codes and store them securely (e.g., in a password manager or a physical safe).
- If you lose your authenticator app and recovery codes, you will need to contact a GitLab administrator to regain access.

---

### **Enforcing MFA for All Users (Admins)**

To enforce MFA for your GitLab instance (if you're an admin):

1. Log in as an admin.
2. Go to **Admin Area** > **Settings** > **General.**
3. Expand **Sign-in Restrictions.**
4. Check the box for **"Require all users to set up Two-Factor Authentication."**
5. Click **Save changes.**

---

### **Testing MFA**

1. Log out and log back into your account.
2. After entering your username and password, GitLab will prompt you for the 6-digit code from your authenticator app.
3. Enter the code to complete the login process.
---

## Next Steps

Once your GitLab instance is configured, proceed to [Basic Usage Guide](USAGE_GUIDE.md) to learn how to create projects, manage repositories, and collaborate with your team.
