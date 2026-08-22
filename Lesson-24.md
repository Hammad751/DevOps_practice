# Lesson 24: Artifact Repository Manager with Nexus

> When software is built, it produces a file — a JAR, a Docker image, an npm package. That file needs to go somewhere secure, versioned, and accessible to the team and the CI/CD pipeline. That "somewhere" is an **Artifact Repository**, and the tool that manages it all is **Nexus**. This lesson covers what artifacts are, why you need a repository manager, how Nexus works, and where it fits into the DevOps workflow.

---

## Table of Contents

1. [What is an Artifact?](#1-what-is-an-artifact)
2. [What is an Artifact Repository?](#2-what-is-an-artifact-repository)
3. [The Problem — Managing Multiple Repositories](#3-the-problem--managing-multiple-repositories)
4. [Nexus Repository Manager](#4-nexus-repository-manager)
5. [Public Repositories vs. Nexus (Private)](#5-public-repositories-vs-nexus-private)
6. [Repository Types in Nexus](#6-repository-types-in-nexus)
7. [Supported Artifact Formats](#7-supported-artifact-formats)
8. [Key Features of Nexus](#8-key-features-of-nexus)
9. [Nexus in the CI/CD Pipeline](#9-nexus-in-the-cicd-pipeline)
10. [Nexus REST API](#10-nexus-rest-api)
11. [Installing Nexus on Linux](#11-installing-nexus-on-linux)
12. [Nexus vs. Other Repository Managers](#12-nexus-vs-other-repository-managers)
13. [Quick Reference Cheat Sheet](#13-quick-reference-cheat-sheet)

---

## 1. What is an Artifact?

An **artifact** is the output of a software build process — the compiled, packaged, deployable version of your application or library.

```
Source Code  ──► BUILD PROCESS ──► Artifact
(what developers               (what gets deployed
 write in IDE)                  to servers)

Java source (.java)  ──► mvn package  ──► myapp-1.0.0.jar
Node.js source       ──► npm pack     ──► myapp-1.0.0.tgz
Python source        ──► pip wheel    ──► myapp-1.0.0.whl
Docker files         ──► docker build ──► myapp:1.0.0 (image)
```

### Common Artifact Formats by Technology

| Technology | Build Tool | Artifact Format | Extension |
|------------|-----------|----------------|-----------|
| Java | Maven | Java Archive | `.jar` |
| Java (web app) | Maven / Gradle | Web Archive | `.war` |
| Java (enterprise) | Maven | Enterprise Archive | `.ear` |
| .NET / C# | MSBuild / NuGet | NuGet Package | `.nupkg` |
| Node.js | npm / yarn / pnpm | npm Package | `.tgz` |
| Python | pip | Python Wheel | `.whl` |
| Ruby | Bundler | Ruby Gem | `.gem` |
| Docker | Docker | Container Image | (layers) |
| Linux | dpkg / rpm | Debian / RPM Package | `.deb` / `.rpm` |
| Any | General | Archive | `.zip` / `.tar.gz` |

### Why Artifacts Matter

Instead of rebuilding from source code on every deployment:

```
WITHOUT artifacts:
  Deploy to staging  → rebuild from source (5 mins) → deploy
  Deploy to prod     → rebuild from source (5 mins) → deploy
  Risk: build might produce different output each time!

WITH artifacts:
  Build once → store artifact → deploy same artifact everywhere
  Deploy to staging → pull artifact (5 sec) → deploy
  Deploy to prod    → pull same artifact    → deploy
  Guarantee: exactly the same binary in every environment
```

> **"Build once, deploy many times"** — this is the core principle. Artifacts ensure that what is tested in staging is exactly what goes to production.

---

## 2. What is an Artifact Repository?

An **artifact repository** is a dedicated storage system for artifacts. It is not just a file share — it provides:

- **Versioning** — stores multiple versions of the same artifact
- **Metadata** — tracks who uploaded what, when, build number, Git commit hash
- **Format support** — understands the specific format (Maven, npm, Docker, etc.)
- **Access control** — controls who can upload and who can download
- **Search** — find artifacts across all repositories
- **Proxying** — cache external public repositories internally

```
CI/CD Pipeline builds artifact
         │
         ▼
  Artifact Repository          ← Team members download
  ┌──────────────────┐         ← Other pipelines pull
  │ myapp-1.0.0.jar  │         ← Staging environment deploys
  │ myapp-1.1.0.jar  │         ← Production environment deploys
  │ myapp-2.0.0.jar  │
  └──────────────────┘
```

---

## 3. The Problem — Managing Multiple Repositories

In a real company, multiple teams use different technology stacks — each producing different artifact formats:

```
┌────────────────────────────────────────────────────────────────┐
│                  MULTI-TEAM COMPANY                            │
│                                                                │
│  Java Team       Python Team     Node.js Team   DevOps Team    │
│  Builds .jar     Builds .whl     Builds .tgz    Docker images  │
│  Uses Maven      Uses pip        Uses npm        registry      │
│                                                                │
│  Without a manager:                                            │
│  4 separate repos → 4 different tools → 4 access systems       │
│  → impossible to manage at scale                               │
│                                                                │
│  With Nexus:                                                   │
│  One tool → all formats → one access system → one backup       │
└────────────────────────────────────────────────────────────────┘
```

---

## 4. Nexus Repository Manager

**Nexus Repository Manager** (by Sonatype) is the industry-standard tool for managing all your artifact repositories in one place.

```
┌──────────────────────────────────────────────────────────────┐
│                  NEXUS REPOSITORY MANAGER                    │
│                                                              │
│  ┌──────────┐  ┌──────────┐  ┌──────────────────┐            │
│  │  Maven   │  │   npm    │  │     Docker       │            │
│  │   Repo   │  │ Registry │  │    Registry      │            │
│  └──────────┘  └──────────┘  └──────────────────┘            │
│  ┌──────────┐  ┌──────────┐  ┌──────────────────┐            │
│  │  NuGet   │  │  PyPI    │  │      Raw         │            │
│  │   Repo   │  │   Repo   │  │      Repo        │            │
│  └──────────┘  └──────────┘  └──────────────────┘            │
│                                                              │
│    Single UI • Single API • Single Access Control            │
└──────────────────────────────────────────────────────────────┘
```

### Two Editions

| Edition | Cost | Best For |
|---------|------|---------|
| **Nexus Repository OSS** | Free, open-source | Most companies — covers all common formats |
| **Nexus Repository Pro** | Paid | Large enterprises needing advanced HA and security scanning |

---

## 5. Public Repositories vs. Nexus (Private)

### Public Repositories

Public repositories host open-source libraries that anyone can download:

| Public Repository | Language | URL |
|------------------|----------|-----|
| **Maven Central** | Java | `repo1.maven.org` |
| **npm Registry** | Node.js | `registry.npmjs.org` |
| **PyPI** | Python | `pypi.org` |
| **Docker Hub** | Docker | `hub.docker.com` |
| **NuGet Gallery** | .NET | `nuget.org` |

### Nexus — Private Repository

Nexus stores **your company's own artifacts**:

```
Public repos: open-source libraries anyone can use
Nexus:        YOUR company's compiled applications and internal libraries
```

### Nexus as a Proxy (Caching Public Repos)

Nexus can also act as a **proxy** for public repositories:

```
Developer/CI requests: spring-boot-starter-web:3.2.0
         │
         ▼
    NEXUS PROXY
    Already cached? ──YES──► Return from Nexus cache (fast)
         │
        NO
         ▼
    Download from Maven Central → Store in Nexus → Return to requester

Benefits:
  - Faster builds (local cache vs internet download)
  - Builds work even if public repo is down
  - Audit and control which external packages are used
```

---

## 6. Repository Types in Nexus

Nexus organizes repositories into three types:

### Hosted Repository

Your **private storage** — where your own built artifacts are uploaded:

```
CI/CD pipeline builds myapp-1.0.0.jar
        ▼ (upload)
Nexus Hosted Repo: maven-releases
        ▼ (download)
Deployment server pulls the artifact
```

- `maven-releases` → stable, production-ready versions (`1.0.0`, `2.3.1`)
- `maven-snapshots` → development/in-progress builds (`1.0.0-SNAPSHOT`)

### Proxy Repository

**Caches external public repositories** inside your network:

```
Nexus Proxy → Maven Central  (caches external JARs)
Nexus Proxy → npm Registry   (caches external npm packages)
Nexus Proxy → Docker Hub     (caches external Docker images)
```

### Group Repository

A **virtual repository** combining multiple hosted and proxy repos into one URL:

```
Group: maven-public
  ├── maven-releases  (hosted)
  ├── maven-snapshots (hosted)
  └── maven-central   (proxy)

CI/CD uses one URL: http://nexus:8081/repository/maven-public/
→ gets everything from all three automatically
```

---

## 7. Supported Artifact Formats

Nexus OSS supports over 20 formats. Most commonly used in DevOps:

| Format | Package Manager | File Type | Used By |
|--------|---------------|-----------|---------|
| **Maven** | Maven / Gradle | `.jar`, `.war`, `.ear` | Java projects |
| **npm** | npm / yarn / pnpm | `.tgz` | Node.js projects |
| **Docker** | Docker / Podman | Image layers | Containerized apps |
| **PyPI** | pip | `.whl`, `.tar.gz` | Python projects |
| **NuGet** | NuGet | `.nupkg` | .NET / C# projects |
| **RubyGems** | gem | `.gem` | Ruby projects |
| **Helm** | Helm | `.tgz` | Kubernetes charts |
| **Raw** | curl / wget | Any file | Scripts, binaries, ZIPs |
| **APT** | apt | `.deb` | Debian/Ubuntu packages |
| **Yum** | yum / dnf | `.rpm` | RHEL/CentOS packages |

---

## 8. Key Features of Nexus

### 1. Multi-Format Support

One tool manages all artifact types — Java, Node.js, Docker, Python, .NET — under a single interface, API, and access control system.

### 2. LDAP Integration

```
Company Active Directory / LDAP
         │
         ▼
    NEXUS (reads users and groups from LDAP)
    → No need to create users manually in Nexus
    → Same company login credentials work
    → When employee leaves AD/LDAP, Nexus access is revoked automatically
```

### 3. Role-Based Access Control (RBAC)

```
Nexus Roles:
  admin           → full access
  developer       → read all repos, upload to snapshots only
  ci-pipeline     → read all repos, upload to releases
  readonly        → download only, no upload

Example:
  jenkins-user    → PUSH to maven-releases
  developer-user  → PULL from maven-public, PUSH to maven-snapshots
  deployer-user   → PULL from maven-releases only
```

### 4. REST API Integration

Nexus is designed for **automation** — not manual work:

```bash
# List all repositories
curl -u admin:password -X GET \
  'http://nexus-host:8081/service/rest/v1/repositories'

# List components in a repository
curl -u admin:password -X GET \
  'http://nexus-host:8081/service/rest/v1/components?repository=maven-releases'

# Search for a specific artifact
curl -u admin:password -X GET \
  'http://nexus-host:8081/service/rest/v1/search?name=myapp&version=1.0.0'

# Delete a component
curl -u admin:password -X DELETE \
  'http://nexus-host:8081/service/rest/v1/components/{id}'
```

### 5. Metadata & Tagging

Each artifact stored in Nexus carries metadata:

```
Artifact: myapp-2.1.0.jar
Metadata:
  - version:      2.1.0
  - buildNumber:  #247
  - gitCommit:    a3f9c12
  - uploadedBy:   jenkins-ci
  - uploadedAt:   2026-08-15 09:30:00
  - size:         18.4 MB
  - checksum MD5: d41d8cd98f00b204e9800998ecf8427e

Tags:
  - release-candidate  → ready for QA testing
  - production-approved → cleared for production
  - deprecated          → old version, stop using
```

### 6. Cleanup Policies

Without cleanup, artifact count grows until disk fills:

```
Day 1:   10 artifacts
Day 30:  300 artifacts
Day 365: 3,650 artifacts  ← disk full → Nexus crashes
```

Cleanup policies automatically delete artifacts matching conditions:

```
Examples:
  - Delete SNAPSHOT versions older than 30 days
  - Delete artifacts not downloaded in 90 days
  - Keep only the last 5 versions of any artifact
  - Delete dev builds (0.0.*) after 7 days
```

### 7. Backup & Restore

```
What Nexus backs up:
  - Repository metadata and search indexes
  - Configuration (repos, users, roles, cleanup policies)
  - Blob stores (the actual artifact files)

Trigger backup via API:
curl -u admin:password -X POST \
  'http://nexus:8081/service/rest/v1/tasks/{task-id}/run'
```

### 8. Search Functionality

```
Search by:
  - Component name   → myapp
  - Version          → 1.0.0 or 2.*
  - Format           → maven, npm, docker
  - Repository       → maven-releases
  - File hash        → verify artifact integrity
  - Upload date range

Via API:
  /service/rest/v1/search?name=myapp&format=maven2
```

### 9. User Token Support

For CI/CD systems (Jenkins, GitHub Actions), using personal credentials is a security risk. User tokens provide a separate credential pair for automated systems:

```
Human user:     admin / mypassword123   (UI login)
CI/CD token:    a3f9c12 / xK9mP2qR      (pipeline calls)

Benefits:
  - Tokens revocable without changing your password
  - Token usage logged separately for auditing
  - Rotating tokens does not disrupt human logins
  - Tokens can be scoped to specific repositories
```

---

## 9. Nexus in the CI/CD Pipeline

```
┌─────────────────────────────────────────────────────────────────┐
│                    CI/CD PIPELINE WITH NEXUS                    │
│                                                                 │
│  1. Developer pushes code to Git                                │
│               │                                                 │
│               ▼                                                 │
│  2. CI pipeline triggers (Jenkins / GitHub Actions)             │
│               │                                                 │
│               ▼                                                 │
│  3. Build tool downloads dependencies                           │
│     Maven / npm / pip → pulls from NEXUS PROXY                  │
│     (Nexus fetches from public repos and caches locally)        │
│               │                                                 │
│               ▼                                                 │
│  4. Application compiled and packaged → myapp-2.1.0.jar         │
│               │                                                 │
│               ▼                                                 │
│  5. CI pipeline UPLOADS artifact to Nexus hosted repo           │
│               │                                                 │
│               ▼                                                 │
│  6. CD pipeline DOWNLOADS artifact from Nexus                   │
│     → deploys to staging server                                 │
│               │                                                 │
│               ▼                                                 │
│  7. After QA approval: deploy same artifact to production       │
│     → guaranteed: same binary tested = same binary deployed     │
└─────────────────────────────────────────────────────────────────┘
```

### Configure Maven to Use Nexus

```xml
<!-- settings.xml -->
<settings>
  <mirrors>
    <mirror>
      <id>nexus</id>
      <mirrorOf>*</mirrorOf>
      <url>http://nexus-host:8081/repository/maven-public/</url>
    </mirror>
  </mirrors>
  <servers>
    <server>
      <id>nexus</id>
      <username>ci-user</username>
      <password>${NEXUS_PASSWORD}</password>
    </server>
  </servers>
</settings>
```

### Configure npm to Use Nexus

```bash
npm config set registry http://nexus-host:8081/repository/npm-proxy/
npm login --registry=http://nexus-host:8081/repository/npm-proxy/
```

### Upload Artifact in Jenkins Pipeline

```groovy
stage('Upload to Nexus') {
    steps {
        nexusArtifactUploader(
            nexusVersion: 'nexus3',
            protocol: 'http',
            nexusUrl: 'nexus-host:8081',
            groupId: 'com.mycompany',
            version: "${BUILD_NUMBER}",
            repository: 'maven-releases',
            credentialsId: 'nexus-credentials',
            artifacts: [[
                artifactId: 'myapp',
                file: "target/myapp-${BUILD_NUMBER}.jar",
                type: 'jar'
            ]]
        )
    }
}
```

---

## 10. Nexus REST API

```bash
BASE_URL="http://nexus-host:8081"
AUTH="-u admin:password"

# List all repositories
curl $AUTH -X GET "$BASE_URL/service/rest/v1/repositories"

# List components in a repository
curl $AUTH -X GET \
  "$BASE_URL/service/rest/v1/components?repository=maven-releases"

# Search for components
curl $AUTH -X GET \
  "$BASE_URL/service/rest/v1/search?name=myapp&version=1.0.0"

# Get a specific component's details
curl $AUTH -X GET \
  "$BASE_URL/service/rest/v1/components/{component-id}"

# Delete a component
curl $AUTH -X DELETE \
  "$BASE_URL/service/rest/v1/components/{component-id}"

# List all assets for a component
curl $AUTH -X GET \
  "$BASE_URL/service/rest/v1/assets?repository=maven-releases"
```

---

## 11. Installing Nexus on Linux

### Option A — Docker (Recommended for Quick Setup)

```bash
# Run Nexus as a Docker container
docker run -d \
  --name nexus \
  -p 8081:8081 \
  -v nexus-data:/nexus-data \
  sonatype/nexus3

# Wait for startup (~1-2 minutes), then check logs
docker logs -f nexus

# Get initial admin password
docker exec nexus cat /nexus-data/admin.password

# Access UI at: http://localhost:8081
```

### Option B — Direct Installation on Linux

```bash
# 1. Install Java (Nexus requires Java 8+)
sudo apt update && sudo apt install -y openjdk-17-jdk

# 2. Create dedicated nexus user (DevOps security practice)
sudo adduser nexus

# 3. Download and extract Nexus
cd /opt
sudo wget https://download.sonatype.com/nexus/3/latest-unix.tar.gz
sudo tar -xzf latest-unix.tar.gz
sudo mv nexus-3* nexus

# 4. Set ownership to nexus user
sudo chown -R nexus:nexus /opt/nexus /opt/sonatype-work

# 5. Configure to run as nexus user
echo 'run_as_user="nexus"' | sudo tee /opt/nexus/bin/nexus.rc

# 6. Create systemd service
sudo tee /etc/systemd/system/nexus.service > /dev/null <<EOF
[Unit]
Description=Nexus Repository Manager
After=network.target

[Service]
Type=forking
LimitNOFILE=65536
ExecStart=/opt/nexus/bin/nexus start
ExecStop=/opt/nexus/bin/nexus stop
User=nexus
Restart=on-abort

[Install]
WantedBy=multi-user.target
EOF

# 7. Start and enable
sudo systemctl daemon-reload
sudo systemctl start nexus
sudo systemctl enable nexus

# 8. Get initial admin password
cat /opt/sonatype-work/nexus3/admin.password

# 9. Access: http://your-server-ip:8081
```

---

## 12. Nexus vs. Other Repository Managers

| Feature | Nexus OSS | JFrog Artifactory | GitHub Packages | AWS CodeArtifact |
|---------|-----------|------------------|----------------|-----------------|
| **Cost** | Free | Paid (Pro) | Free (with GitHub) | Pay per use |
| **Self-hosted** | Yes | Yes | No | No |
| **Format support** | 20+ | 30+ | Limited | Limited |
| **Docker registry** | Yes | Yes | Yes | No |
| **LDAP integration** | Yes | Yes | No | IAM |
| **REST API** | Yes | Yes | Yes | Yes |
| **Proxy/cache** | Yes | Yes | No | Yes |
| **Best for** | Self-hosted teams | Enterprise | GitHub-centric | AWS-native |

---

## 13. Quick Reference Cheat Sheet

### Core Concepts

```
Artifact        = built, deployable file (JAR, Docker image, npm package)
Repository      = storage for artifacts of a specific format
Nexus           = manager for ALL repository types in one tool

Repository types:
  Hosted  = your private storage (CI/CD uploads here)
  Proxy   = caches external public repos (Maven Central, npm)
  Group   = combines hosted + proxy into one URL
```

### Key Features Summary

| Feature | Purpose |
|---------|---------|
| Multi-format | One tool for Maven, npm, Docker, PyPI, NuGet, Helm |
| LDAP integration | Use company AD credentials |
| RBAC | Control upload/download per repository per user |
| REST API | Automate everything from CI/CD pipelines |
| Proxy/caching | Cache public repos locally — faster and resilient |
| Metadata & tagging | Track build, commit, version per artifact |
| Cleanup policies | Auto-delete old artifacts, prevent disk overflow |
| Backup & restore | Protect artifact history and configuration |
| Search | Find any artifact across all repos instantly |
| User tokens | Secure separate credentials for CI systems |

### CI/CD Flow with Nexus

```
Code push
  → CI builds app
  → Downloads deps from Nexus PROXY (cached external libs)
  → Packages artifact
  → Uploads to Nexus HOSTED repo
  → CD pulls artifact from Nexus
  → Deploys to server
```

### Default Access

```
URL:  http://your-server:8081
User: admin
Pass: (from admin.password file on first login)
```

---

*End of Lesson 24*