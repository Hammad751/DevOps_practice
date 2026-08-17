# Lesson 21: Git for DevOps Engineers

> Git is not just a developer tool — it is the **central nervous system of DevOps**. Every piece of infrastructure code, every deployment script, every CI/CD pipeline, and every configuration file lives in Git. As a DevOps engineer, you use Git differently from a developer: you manage infrastructure repositories, own branching strategies, integrate automation tools with code repos, and treat config files with the same discipline as application code.

---

## Table of Contents

1. [Why Git Matters in DevOps](#1-why-git-matters-in-devops)
2. [Use Case 1 — Infrastructure as Code (IaC) Version Control](#2-use-case-1--infrastructure-as-code-iac-version-control)
3. [Use Case 2 — CI/CD Pipeline Integration](#3-use-case-2--cicd-pipeline-integration)
4. [Git Core Concepts](#4-git-core-concepts)
5. [Essential Git Commands](#5-essential-git-commands)
6. [Branching Strategies for DevOps](#6-branching-strategies-for-devops)
7. [Pull Requests & Code Review Workflow](#7-pull-requests--code-review-workflow)
8. [Git Tags & Release Versioning](#8-git-tags--release-versioning)
9. [Git Hooks — Automation at the Repo Level](#9-git-hooks--automation-at-the-repo-level)
10. [Connecting Git to CI/CD Tools](#10-connecting-git-to-cicd-tools)
11. [Git Best Practices for DevOps](#11-git-best-practices-for-devops)
12. [Quick Reference Cheat Sheet](#12-quick-reference-cheat-sheet)

---

## 1. Why Git Matters in DevOps

In DevOps, **everything is code** — and everything that is code lives in Git.

```
┌────────────────────────────────────────────────────────────────┐
│                   WHAT DEVOPS STORES IN GIT                    │
│                                                                │
│  Application Code       Infrastructure Code (IaC)              │
│  ─────────────────       ─────────────────────────             │
│  • Source code           • Terraform configs (.tf)             │
│  • Unit tests            • Ansible playbooks (.yml)            │
│  • Dockerfiles           • Kubernetes manifests (.yaml)        │
│  • docker-compose.yml    • Helm charts                         │
│                          • CI/CD pipeline definitions          │
│                          • Bash / Python scripts               │
│                          • Environment configs                 │
└────────────────────────────────────────────────────────────────┘
```

### The DevOps Engineer's Git Responsibilities

Unlike developers who primarily commit application code, DevOps engineers:

| Responsibility | Description |
|---------------|-------------|
| **Own the repos** | Manage repository structure, access control, branch protection rules |
| **Design branching strategy** | Define how code flows from dev → staging → production |
| **Integrate automation tools** | Connect Jenkins, GitHub Actions, ArgoCD to repos |
| **Manage IaC version history** | Track every change to infrastructure configs |
| **Enforce Git best practices** | Commit standards, PR requirements, code review gates |
| **Handle releases & versioning** | Git tags, semantic versioning, release branches |

---

## 2. Use Case 1 — Infrastructure as Code (IaC) Version Control

### The Problem Git Solves for IaC

Before Git, infrastructure changes were made manually — directly on servers. This caused:

```
Engineer A changes nginx config on server
    → No record of what changed or why
    → Engineer B overwrites it a week later
    → Production breaks
    → Nobody knows what the correct config was
```

With Git, every infrastructure change is tracked:

```
Engineer A changes nginx config
    → Commits with message: "fix: increase worker_connections to 1024 for traffic spike"
    → Opens PR → team reviews
    → Merged → automatically deployed
    → Full history preserved forever
```

### What IaC Files Live in Git

```
infra-repo/
├── terraform/
│   ├── main.tf              ← Cloud resource definitions
│   ├── variables.tf         ← Input variables
│   └── outputs.tf           ← Output values
├── ansible/
│   ├── playbooks/
│   │   ├── setup-server.yml ← Server provisioning
│   │   └── deploy-app.yml   ← Application deployment
│   └── inventory/
│       ├── production       ← Production server list
│       └── staging          ← Staging server list
├── kubernetes/
│   ├── deployments/
│   │   └── app-deployment.yaml
│   ├── services/
│   │   └── app-service.yaml
│   └── configmaps/
│       └── app-config.yaml
├── scripts/
│   ├── backup.sh            ← Bash automation scripts
│   └── health-check.py     ← Python monitoring scripts
└── .github/
    └── workflows/
        └── deploy.yml       ← CI/CD pipeline definition
```

### Why Version Control for IaC is Critical

| Benefit | Description |
|---------|-------------|
| **Change history** | Know exactly what changed, when, and who changed it |
| **Rollback** | Instantly revert to a previous working infrastructure state |
| **Collaboration** | Multiple engineers work on infra without overwriting each other |
| **Review process** | Infrastructure changes go through PR review like code |
| **Audit trail** | Compliance and security audits require change records |
| **Disaster recovery** | Rebuild entire infrastructure from repo if servers are lost |

> **Key principle:** Treat infrastructure code with the same discipline as application code. Every change goes through Git — never directly modify production configs manually.

---

## 3. Use Case 2 — CI/CD Pipeline Integration

### What CI/CD Means

```
CI — Continuous Integration:
     Every code push automatically triggers:
     → Build the application
     → Run automated tests
     → Report results to the team
     Fast feedback: catch bugs within minutes, not days.

CD — Continuous Delivery/Deployment:
     After CI passes, automatically:
     → Package the application (Docker image, artifact)
     → Deploy to staging (Continuous Delivery)
     → Deploy to production (Continuous Deployment)
```

### How Git Triggers CI/CD

Git is the **starting point** of every CI/CD pipeline. The automation tool watches the repository for events:

```
Developer pushes code to GitHub
           │
           ▼
    Git event triggered
    (push / pull_request / tag)
           │
           ▼
  CI/CD tool receives webhook
  (GitHub Actions / Jenkins / GitLab CI)
           │
           ▼
  Pipeline starts automatically:
    1. Checkout code from repo
    2. Install dependencies
    3. Run tests
    4. Build Docker image
    5. Push to container registry
    6. Deploy to server/Kubernetes
           │
           ▼
  Result reported back to the PR/commit
  ✅ All checks passed → safe to merge
  ❌ Tests failed → block the merge
```

### The DevOps Engineer's Role in CI/CD

DevOps engineers are responsible for **building and maintaining** the automation that connects Git to deployment:

1. **Set up the CI/CD tool** (Jenkins, GitHub Actions, GitLab CI, CircleCI)
2. **Write the pipeline definition** (YAML files stored in the Git repo itself)
3. **Integrate with the application repo** — configure webhooks or native integrations
4. **Define what triggers the pipeline** — push to main, PR opened, tag created
5. **Manage secrets** — store credentials safely (not in Git) for deployments
6. **Monitor pipeline health** — fix broken pipelines, optimize slow stages

### Example: GitHub Actions Pipeline (stored in the repo)

```yaml
# .github/workflows/deploy.yml
name: CI/CD Pipeline

on:
  push:
    branches: [ "main" ]        # Triggers on push to main
  pull_request:
    branches: [ "main" ]        # Triggers on PR to main

jobs:
  build-and-test:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Set up Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '22'

      - name: Install dependencies
        run: npm install

      - name: Run tests
        run: npm test

      - name: Build Docker image
        run: docker build -t myapp:${{ github.sha }} .

      - name: Push to Docker Hub
        run: |
          docker login -u ${{ secrets.DOCKER_USERNAME }} \
                       -p ${{ secrets.DOCKER_PASSWORD }}
          docker push myapp:${{ github.sha }}

  deploy:
    needs: build-and-test        # Only runs if build-and-test passes
    runs-on: ubuntu-latest

    steps:
      - name: Deploy to server
        run: |
          ssh user@server "docker pull myapp:${{ github.sha }} && \
                           docker restart my-container"
```

---

## 4. Git Core Concepts

### The Three States of Git

```
┌──────────────────────────────────────────────────────────────┐
│                    GIT STATES                                │
│                                                              │
│  Working Directory    Staging Area        Repository         │
│  ─────────────────    ────────────        ──────────         │
│  Files you edit       Files ready         Committed          │
│  (untracked /         to be committed     snapshots          │
│   modified)           (git add)           (git commit)       │
│                                                              │
│  edit file.txt  ──►  git add file.txt  ──► git commit        │
└──────────────────────────────────────────────────────────────┘
```

### Key Concepts

| Concept | Description |
|---------|-------------|
| **Repository (repo)** | The project folder tracked by Git — contains full history |
| **Commit** | A snapshot of changes at a point in time, with a unique hash (SHA) |
| **Branch** | A parallel line of development — isolates changes from main code |
| **Merge** | Combines one branch into another |
| **Remote** | A copy of the repo hosted externally (GitHub, GitLab, Bitbucket) |
| **Clone** | Download a complete copy of a remote repo |
| **Pull** | Fetch + merge remote changes into your local branch |
| **Push** | Upload your local commits to the remote repo |
| **Pull Request (PR)** | A request to merge a branch — triggers review and CI checks |
| **HEAD** | Pointer to the current commit you are working from |

---

## 5. Essential Git Commands

### Setup

```bash
# Configure identity (required before first commit)
git config --global user.name "Your Name"
git config --global user.email "you@example.com"

# Set default branch name to main
git config --global init.defaultBranch main

# Set preferred editor
git config --global core.editor vim

# View all configuration
git config --list
```

### Starting a Repository

```bash
# Initialize a new repo in current directory
git init

# Clone an existing remote repo
git clone https://github.com/org/repo.git

# Clone into a specific folder
git clone https://github.com/org/repo.git my-project
```

### Daily Workflow

```bash
# Check status of working directory
git status

# See what changed (unstaged)
git diff

# Stage specific file
git add filename.tf

# Stage all changes
git add .

# Commit staged changes
git commit -m "feat: add nginx deployment manifest"

# Stage and commit in one step (tracked files only)
git commit -am "fix: correct replica count"

# View commit history
git log
git log --oneline          # compact view
git log --oneline --graph  # visual branch graph
```

### Branching

```bash
# List all branches
git branch
git branch -a              # includes remote branches

# Create a new branch
git branch feature/add-monitoring

# Switch to a branch
git checkout feature/add-monitoring

# Create and switch in one command
git checkout -b feature/add-monitoring

# Modern syntax (Git 2.23+)
git switch -c feature/add-monitoring

# Delete a branch (after merging)
git branch -d feature/add-monitoring

# Force delete (unmerged branch)
git branch -D feature/add-monitoring
```

### Remote Operations

```bash
# View remotes
git remote -v

# Add a remote
git remote add origin https://github.com/user/repo.git

# Push branch to remote
git push origin feature/add-monitoring

# Push and set upstream (track remote branch)
git push -u origin feature/add-monitoring

# Pull latest changes
git pull origin main

# Fetch without merging
git fetch origin
```

### Merging & Rebasing

```bash
# Merge a branch into current branch
git merge feature/add-monitoring

# Rebase current branch onto main
git rebase main

# Interactive rebase (squash, reorder commits)
git rebase -i HEAD~3       # last 3 commits
```

### Undoing Changes

```bash
# Unstage a file (keep changes)
git restore --staged filename.tf

# Discard changes in working directory
git restore filename.tf

# Undo last commit (keep changes staged)
git reset --soft HEAD~1

# Undo last commit (keep changes unstaged)
git reset --mixed HEAD~1

# Undo last commit (discard changes — DESTRUCTIVE)
git reset --hard HEAD~1

# Revert a commit (safe — creates new commit)
git revert <commit-hash>   # preferred in production
```

> **Production rule:** Use `git revert` (not `git reset`) to undo changes in shared branches. `revert` preserves history and is safe for teams; `reset --hard` rewrites history and causes problems for anyone who pulled the commits.

### Stashing

```bash
# Save work in progress without committing
git stash

# Stash with a description
git stash save "WIP: adding health check endpoint"

# List all stashes
git stash list

# Apply most recent stash (keeps it in stash list)
git stash apply

# Apply and remove from stash list
git stash pop

# Apply a specific stash
git stash apply stash@{2}
```

---

## 6. Branching Strategies for DevOps

A **branching strategy** defines how your team uses branches to manage code flow from development to production. DevOps engineers choose and enforce these strategies.

### Strategy 1 — GitFlow

Best for: Teams with scheduled releases and formal QA cycles.

```
main ──────────────────────────────────────────────► (production)
  │                                     ▲
  └─► develop ───────────────────────► merge
           │                ▲
           └─► feature/x ──┘
           └─► feature/y ──┘
           └─► release/1.2 ──► (QA + fixes) ──► main
                                    │
                               hotfix/critical ──► main + develop
```

**Branch types:**
| Branch | Purpose |
|--------|---------|
| `main` | Production-ready code only |
| `develop` | Integration branch — all features merge here |
| `feature/*` | Individual feature development |
| `release/*` | Pre-release stabilization and QA |
| `hotfix/*` | Urgent production fixes |

### Strategy 2 — GitHub Flow

Best for: Teams deploying frequently (SaaS, web apps). Simpler than GitFlow.

```
main ────────────────────────────────────────────────►
  │          ▲          ▲          ▲
  └─feature──┘  └─fix───┘  └─feat──┘
     (PR → CI → review → merge → auto-deploy)
```

**Rules:**
- `main` is always deployable
- All work happens on feature branches
- Every PR triggers CI before merging
- Merge to main = deploy to production

### Strategy 3 — Trunk-Based Development

Best for: High-velocity teams (Google, Facebook, Netflix). Maximum CI/CD speed.

```
main (trunk) ─────────────────────────────────────────►
  │   ▲   │   ▲   │   ▲
  └───┘   └───┘   └───┘
  (short-lived branches, < 2 days, merged frequently)
```

**Rules:**
- All developers commit to `main` multiple times per day
- Feature branches exist for at most 1–2 days
- Feature flags hide incomplete features until ready
- Maximizes CI effectiveness — catches integration bugs immediately

### Choosing a Strategy

| Factor | GitFlow | GitHub Flow | Trunk-Based |
|--------|---------|-------------|-------------|
| Release frequency | Scheduled (weekly/monthly) | Frequent (daily) | Continuous |
| Team size | Medium-large | Any | Any |
| CI/CD maturity | Basic | Intermediate | Advanced |
| Complexity | High | Low | Low |
| Used by | Enterprise software | SaaS products | Google, Netflix |

---

## 7. Pull Requests & Code Review Workflow

A **Pull Request (PR)** is a request to merge one branch into another. It is the heart of DevOps collaboration — every change to infrastructure or application code goes through a PR.

### The Complete PR Workflow

```
1. Create feature branch
   git checkout -b feature/add-prometheus-monitoring

2. Make changes and commit
   git add monitoring/prometheus.yaml
   git commit -m "feat: add Prometheus scrape config for app metrics"

3. Push to remote
   git push -u origin feature/add-prometheus-monitoring

4. Open PR on GitHub/GitLab
   → Title and description explain the change
   → Link to the ticket/issue

5. CI pipeline runs automatically
   → Linting checks
   → Terraform plan (for IaC)
   → Security scan
   → Tests

6. Team reviews the code
   → Comment on specific lines
   → Request changes or approve

7. Address review feedback
   → Push new commits to the same branch
   → CI runs again

8. Merge (after approvals + CI passes)
   → Feature branch deleted
   → Changes deployed automatically (CD)
```

### PR Best Practices for DevOps

```bash
# Keep PRs small and focused — one change per PR
# Bad:  "Update nginx, add monitoring, fix firewall rules, upgrade Node"
# Good: "feat: add Prometheus monitoring for nginx"

# Write meaningful commit messages using Conventional Commits:
git commit -m "feat: add health check endpoint to nginx config"
git commit -m "fix: correct replica count in deployment manifest"
git commit -m "chore: update Terraform provider versions"
git commit -m "docs: add README for Ansible playbooks"
```

### Conventional Commits Format

```
<type>(<scope>): <description>

Types:
feat     → new feature
fix      → bug fix
chore    → maintenance (no production change)
docs     → documentation only
refactor → code restructure (no functional change)
test     → adding or updating tests
ci       → CI/CD pipeline changes
infra    → infrastructure changes
```

---

## 8. Git Tags & Release Versioning

Tags mark specific commits as significant — typically used to mark production releases.

### Semantic Versioning (SemVer)

```
v MAJOR . MINOR . PATCH
  ─────   ─────   ─────
    │       │       │
    │       │       └── Bug fixes, no breaking changes
    │       └────────── New features, backward compatible
    └────────────────── Breaking changes
```

Examples:
- `v1.0.0` → first production release
- `v1.1.0` → new feature added
- `v1.1.1` → bug fix
- `v2.0.0` → breaking change (API redesign, major upgrade)

### Working with Tags

```bash
# Create a lightweight tag
git tag v1.0.0

# Create an annotated tag (recommended — includes message)
git tag -a v1.0.0 -m "Release: initial production deployment"

# Tag a specific past commit
git tag -a v1.0.0 <commit-hash> -m "Release: initial production deployment"

# List all tags
git tag

# Push a single tag to remote
git push origin v1.0.0

# Push all tags at once
git push origin --tags

# Delete a tag locally
git tag -d v1.0.0

# Delete a tag on remote
git push origin --delete v1.0.0
```

### Tags as CI/CD Triggers

In most pipelines, creating a tag automatically triggers a production deployment:

```yaml
# GitHub Actions — trigger deployment only on tagged release
on:
  push:
    tags:
      - 'v*'           # Triggers on any tag starting with 'v'

jobs:
  deploy-production:
    runs-on: ubuntu-latest
    steps:
      - name: Deploy release ${{ github.ref_name }}
        run: ./deploy.sh production ${{ github.ref_name }}
```

```bash
# Create and push a release tag → triggers production deployment
git tag -a v2.1.0 -m "Release: add auto-scaling support"
git push origin v2.1.0
```

---

## 9. Git Hooks — Automation at the Repo Level

**Git hooks** are scripts that run automatically at specific points in the Git workflow. They allow you to enforce standards and automate checks locally — before code even reaches the CI pipeline.

```
git commit
    │
    ▼
pre-commit hook      ← runs BEFORE commit is created
    │  (lint check, format check, secret scan)
    │
    ▼ (if hook passes)
commit-msg hook      ← validates commit message format
    │
    ▼ (if hook passes)
commit created
    │
git push
    │
    ▼
pre-push hook        ← runs BEFORE push to remote
    │  (run tests, final checks)
    ▼
code pushed to remote
```

### Common Hook Use Cases

| Hook | When It Runs | Common Uses |
|------|-------------|-------------|
| `pre-commit` | Before creating a commit | Linting, formatting, secret scanning |
| `commit-msg` | After commit message written | Enforce Conventional Commits format |
| `pre-push` | Before pushing to remote | Run full test suite |
| `post-receive` | After receiving a push (server-side) | Trigger deployments |

### Example: pre-commit Hook (Secret Scanning)

```bash
# .git/hooks/pre-commit
#!/bin/bash

# Scan for accidentally committed secrets
if git diff --cached | grep -E "(password|secret|api_key|AWS_SECRET)" -i; then
    echo "ERROR: Potential secret detected in staged files."
    echo "Remove the secret before committing."
    exit 1
fi

echo "Secret scan passed."
exit 0
```

```bash
# Make the hook executable
chmod +x .git/hooks/pre-commit
```

> **Never commit secrets to Git.** Use environment variables, `.env` files (added to `.gitignore`), or secret managers (AWS Secrets Manager, HashiCorp Vault). Once a secret is committed, assume it is compromised — even if you delete it, it lives in the Git history.

---

## 10. Connecting Git to CI/CD Tools

### How CI/CD Tools Integrate with Git Repos

Most CI/CD tools connect to Git repositories via **webhooks** — the Git platform sends an HTTP request to the CI/CD tool whenever a specified event occurs (push, PR, tag).

```
GitHub Repo                    CI/CD Tool
───────────                    ──────────
Developer pushes  ──webhook──► Jenkins / GitHub Actions / GitLab CI
                               │
                               ▼
                          Pipeline starts:
                          - Checkout code
                          - Build
                          - Test
                          - Deploy
                               │
                               ▼
                          Status reported back to PR
                          ✅ or ❌
```

### GitHub Actions (Native — No Setup Required)

```yaml
# Just create .github/workflows/pipeline.yml in your repo
# GitHub automatically detects and runs it
on:
  push:
    branches: [ main ]
```

### Jenkins (Self-Hosted)

```groovy
// Jenkinsfile — stored in the root of your repo
pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/org/repo.git'
            }
        }
        stage('Build') {
            steps {
                sh 'docker build -t myapp:${BUILD_NUMBER} .'
            }
        }
        stage('Test') {
            steps {
                sh 'npm test'
            }
        }
        stage('Deploy') {
            steps {
                sh './deploy.sh production'
            }
        }
    }
}
```

### GitLab CI

```yaml
# .gitlab-ci.yml — stored in your repo root
stages:
  - build
  - test
  - deploy

build:
  stage: build
  script:
    - docker build -t myapp:$CI_COMMIT_SHA .

test:
  stage: test
  script:
    - npm test

deploy:
  stage: deploy
  script:
    - ./deploy.sh production
  only:
    - main
```

---

## 11. Git Best Practices for DevOps

### The `.gitignore` File

Never commit files that should not be in version control:

```bash
# Create .gitignore in repo root
vim .gitignore
```

```
# Secrets and credentials
.env
*.pem
*.key
*secret*
credentials.json

# Terraform state (contains sensitive data)
*.tfstate
*.tfstate.backup
.terraform/

# OS and editor files
.DS_Store
.idea/
*.swp

# Node modules
node_modules/

# Build outputs
dist/
build/
*.log
```

### Commit Message Standards

```bash
# Bad commit messages:
git commit -m "fix"
git commit -m "changes"
git commit -m "WIP"
git commit -m "asdfghjkl"

# Good commit messages (Conventional Commits):
git commit -m "feat(k8s): add horizontal pod autoscaler for api service"
git commit -m "fix(nginx): increase proxy timeout to prevent 504 errors"
git commit -m "ci: add security scanning stage to deployment pipeline"
git commit -m "infra: upgrade Terraform AWS provider to v5.0"
```

### Branch Naming Conventions

```bash
feature/short-description      # new features
fix/issue-description          # bug fixes
hotfix/critical-issue          # urgent production fixes
release/v1.2.0                 # release preparation
chore/update-dependencies      # maintenance tasks
infra/add-monitoring-stack     # infrastructure changes
ci/optimize-build-pipeline     # CI/CD changes
```

### Security Best Practices

```bash
# Scan repo history for accidentally committed secrets
git log -p | grep -i "password\|secret\|api_key"

# Remove a file from all history (nuclear option — use carefully)
git filter-branch --force --index-filter \
  'git rm --cached --ignore-unmatch path/to/secret.txt' \
  --prune-empty --tag-name-filter cat -- --all

# Use BFG Repo Cleaner (faster and safer)
java -jar bfg.jar --delete-files secret.txt

# Rotate any exposed credentials IMMEDIATELY
# Do not rely on deletion alone — assume it was seen
```

---

## 12. Quick Reference Cheat Sheet

### Setup

```bash
git config --global user.name "Name"
git config --global user.email "email"
git init                          # new repo
git clone <url>                   # clone repo
```

### Daily Workflow

```bash
git status                        # check state
git add .                         # stage all
git commit -m "message"           # commit
git push origin <branch>          # push
git pull origin main              # pull latest
```

### Branching

```bash
git checkout -b feature/name      # create + switch
git switch -c feature/name        # modern syntax
git branch -d feature/name        # delete branch
git merge feature/name            # merge into current
git rebase main                   # rebase onto main
```

### Undoing (Safe)

```bash
git restore --staged file         # unstage
git restore file                  # discard changes
git revert <hash>                 # undo commit safely
git stash                         # save WIP
git stash pop                     # restore WIP
```

### Tags & Releases

```bash
git tag -a v1.0.0 -m "Release"   # create tag
git push origin v1.0.0            # push tag
git push origin --tags            # push all tags
git tag                           # list tags
```

### Remote

```bash
git remote -v                     # list remotes
git fetch origin                  # fetch without merge
git push -u origin <branch>       # push + set upstream
```

### Inspection

```bash
git log --oneline --graph         # visual history
git diff                          # unstaged changes
git diff --staged                 # staged changes
git show <hash>                   # show a commit
git blame filename                # who changed what line
```

### CI/CD Integration Summary

```
Push to feature branch → CI runs tests
Open PR              → CI runs + team reviews
Merge to main        → CD deploys to staging
Create tag (v1.x.x)  → CD deploys to production
```

---

*End of Lesson 21*