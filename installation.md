# Installation Guide

This guide provides step-by-step instructions to install a GitLab instance. We will cover three popular methods:

1. [Installing GitLab using Docker](#installing-gitlab-using-docker)
2. [Installing GitLab using the Omnibus package](#installing-gitlab-using-the-omnibus-package)
3. [Installing GitLab from source](#installing-gitlab-from-source)

---

## Installing GitLab using Docker

Docker is a popular way to set up GitLab as it simplifies dependency management. Follow these steps:

1. **Install Docker**
   - Follow the official Docker installation guide for your OS: [Docker Installation Docs](https://docs.docker.com/get-docker/).

2. **Pull the GitLab Docker Image**
```bash
docker pull gitlab/gitlab-ce:latest
```

3. **Run the Docker Container**
```bash
docker run --detach \
  --hostname gitlab.example.com \
  --publish 443:443 --publish 80:80 --publish 22:22 \
  --name gitlab \
  --restart always \
  --volume /srv/gitlab/config:/etc/gitlab \
  --volume /srv/gitlab/logs:/var/log/gitlab \
  --volume /srv/gitlab/data:/var/opt/gitlab \
  gitlab/gitlab-ce:latest
```
   Replace `gitlab.example.com` with your server's domain or IP address.

---

## Installing GitLab using the Omnibus Package (preferred):

The Omnibus package is the easiest way to install GitLab on a Linux server. It includes everything you need in a single installation.

1. **Install Dependencies

   - Update your system:
```bash
sudo apt update && sudo apt upgrade -y
```

   - Install required packages:
```bash
sudo apt install -y curl openssh-server ca-certificates
```

   - Install Postfix for email notifications (optional):
```bash
sudo apt install -y postfix
```
During installation, select "Internet Site" and configure your domain.

2. **Add the GitLab Repository**

```bash
curl -s https://packages.gitlab.com/install/repositories/gitlab/gitlab-ce/script.deb.sh | sudo bash
```

3. **Install GitLab**

```bash
sudo apt install gitlab-ce
```

4. **Configure GitLab**

   - Edit the configuration file:
```bash
sudo nano /etc/gitlab/gitlab.rb
```
Set `external_url` to your server's domain or IP address.
 
   - Reconfigure GitLab:
```bash
sudo gitlab-ctl reconfigure
```

---

## Installing GitLab from Source

Installing from source is the most customizable option but requires more setup. This is recommended for advanced users.

1. **Install Dependencies**
   ```bash
   sudo apt update && sudo apt install -y \
     curl openssh-server ca-certificates tzdata \
     perl build-essential zlib1g-dev \
     libyaml-dev libssl-dev libgdbm-dev \
     libreadline-dev libncurses5-dev libffi-dev \
     redis-server libxml2-dev libxslt-dev libpq-dev \
     libicu-dev logrotate python3 python3-pip
   ```

2. **Clone the GitLab Repository**
   ```bash
   git clone https://gitlab.com/gitlab-org/gitlab-foss.git
   cd gitlab-foss
   ```

3. **Install Ruby and Bundler**
   ```bash
   sudo apt install -y ruby ruby-bundler
   ```

4. **Install Gems and Configure GitLab**
   ```bash
   bundle install --deployment --without development test
   cp config/gitlab.yml.example config/gitlab.yml
   ```
   Modify `gitlab.yml` to suit your environment.

5. **Start GitLab**
   ```bash
   bundle exec rails s -e production
   ```

---

## Next Steps

Once GitLab is installed, proceed to [Configuration Guide](CONFIGURATION.md) to set up your instance.
