# 📚 **GitLab Usage Guide**

This guide is designed to help you get started with the basics of using GitLab after installation and configuration. Whether you're creating projects, collaborating with teams, or setting up CI/CD pipelines, this guide will provide you with the foundation you need.

---

## 📋 **Table of Contents**

1. [Creating Projects and Repositories](#creating-projects-and-repositories)
2. [Using Git with GitLab](#using-git-with-gitlab)
3. [Managing Members and Permissions](#managing-members-and-permissions)
4. [Working with Issues and Merge Requests](#working-with-issues-and-merge-requests)
5. [Setting Up CI/CD Pipelines](#setting-up-cicd-pipelines)

---

## 1️⃣ **Creating Projects and Repositories**

1. **Log in to GitLab** and navigate to the dashboard.
2. Click **“New Project”** to create a new repository.
    - Choose between:
        - **Blank Project**: Start from scratch.
        - **Import Project**: Import from another version control system like GitHub or Bitbucket.
    - Fill in the **project name**, **description**, and choose its **visibility level** (private, internal, or public).
3. Click **“Create Project”** to initialize the repository.

---

## 2️⃣ **Using Git with GitLab**

1. Clone your repository to your local machine:
```bash
git clone https://<your-gitlab-instance>/<username>/<project-name>.git
```

2. Make changes to your files and stage them:
```bash
git add <file-name>
```

3. Commit your changes with a meaningful message:
```bash
git commit -m "Your commit message"
```

4. Push changes to the GitLab repository:
```bash
git push origin main
```

---

## 3️⃣ **Managing Members and Permissions**

1. Go to the **project settings**.
2. Click **“Members”** in the sidebar.
3. Add team members by their username or email address.
4. Assign them roles:
    - **Guest**: Read-only access to the project.
    - **Reporter**: Can read code and issues.
    - **Developer**: Can contribute to code and issues.
    - **Maintainer**: Full access to manage the project.
    - **Owner**: Reserved for top-level access.
    - **Planner**: A user role that can create and manage issues and milestones but cannot modify the code or merge requests.

---

## 4️⃣ **Working with Issues and Merge Requests**

- **Issues**:
    - Navigate to the **“Issues”** tab to create, track, and manage tasks or bugs.
    - Assign issues to team members and set milestones for deadlines.
- **Merge Requests**:
    - Create a merge request to propose changes to the codebase.
    - Include a description of the changes and request a code review from teammates.

---

## 5️⃣ **Setting Up CI/CD Pipelines**

1. **Create a `.gitlab-ci.yml` file** in your repository's root directory:
```yaml
stages:
  - build
  - test

build-job:
  stage: build
  script:
    - echo "Compiling the application..."

test-job:
  stage: test
  script:
    - echo "Running tests..."
```

2. Push the `.gitlab-ci.yml` file to the repository:
```bash
git add .gitlab-ci.yml
git commit -m "Add CI/CD pipeline"
git push origin main
```

3. Monitor the pipeline execution under the **CI/CD > Pipelines** tab in the GitLab interface.

---
