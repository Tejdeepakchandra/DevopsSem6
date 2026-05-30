# Jenkins Architecture & Foundations

## What is Jenkins?

Jenkins is an **open-source automation server** used for Continuous Integration and Continuous Delivery (CI/CD). It is the most widely used CI/CD tool in the industry.

---

## Jenkins Architecture (Master/Agent Model)

```
                    ┌─────────────────────┐
                    │   Jenkins Master    │
                    │  (Controller Node)  │
                    │                     │
                    │  - Scheduling jobs  │
                    │  - Managing agents  │
                    │  - Serving UI       │
                    │  - Plugin mgmt     │
                    └────────┬────────────┘
                             │
              ┌──────────────┼──────────────┐
              │              │              │
      ┌───────▼──────┐ ┌────▼────────┐ ┌───▼──────────┐
      │  Agent 1     │ │  Agent 2    │ │  Agent 3     │
      │  (Linux)     │ │  (Windows)  │ │  (Docker)    │
      │              │ │             │ │              │
      │ Executes     │ │ Executes    │ │ Executes     │
      │ build jobs   │ │ build jobs  │ │ build jobs   │
      └──────────────┘ └─────────────┘ └──────────────┘
```

### Master (Controller):
- Manages the entire Jenkins environment
- Schedules and distributes jobs to agents
- Serves the web UI
- Manages plugins and configuration
- Stores build history and artifacts

### Agents (formerly "Slaves"):
- Execute build jobs assigned by the master
- Can run on different OS/platforms
- Can be added/removed dynamically
- Types:
  - **SSH agents**: Connected via SSH
  - **JNLP agents**: Java-based connection
  - **Docker agents**: Containers as agents
  - **Cloud agents**: Spun up on demand (AWS, Azure, GCP)

---

## Installation & Setup

### Docker Installation (Recommended):
```bash
docker run -d \
  --name jenkins \
  -p 8080:8080 \
  -p 50000:50000 \
  -v jenkins_home:/var/jenkins_home \
  jenkins/jenkins:lts
```

### Getting Initial Admin Password:
```bash
docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword
```

### Post-Installation Steps:
1. Access Jenkins at `http://localhost:8080`
2. Enter the initial admin password
3. Install suggested plugins
4. Create the admin user
5. Configure Jenkins URL

---

## UI Overview

Jenkins provides a web-based interface:
- **Dashboard**: Overview of all jobs and their status
- **New Item**: Create new jobs/pipelines
- **Manage Jenkins**: System configuration, plugins, security
- **Build History**: View past builds across all jobs
- **Nodes**: Manage agents and executors

---

## Plugin Management

Jenkins functionality is extended through **plugins** (1,800+ available).

### Essential Plugins:
| Plugin | Purpose |
|--------|---------|
| Git | Git SCM integration |
| Pipeline | Declarative/Scripted pipelines |
| Docker | Docker build and publish |
| Maven Integration | Maven project support |
| Blue Ocean | Modern UI for pipelines |
| Credentials | Secure credential management |
| GitHub Branch Source | GitHub integration |

### Managing Plugins:
- **Install**: Manage Jenkins → Plugins → Available plugins
- **Update**: Manage Jenkins → Plugins → Updates
- **Uninstall**: Manage Jenkins → Plugins → Installed → Uninstall

---

## Security, Users & Roles

### Security Configuration:
```
Manage Jenkins → Security
├── Authentication
│   ├── Jenkins' own user database
│   ├── LDAP
│   └── Active Directory
├── Authorization
│   ├── Matrix-based security
│   ├── Role-based (with Role Strategy plugin)
│   └── Project-based
└── CSRF Protection (enabled by default)
```

### Role-Based Access Control (RBAC):
```
Roles:
├── Admin       → Full access
├── Developer   → Build, configure own jobs
├── Viewer      → Read-only access
└── Deploy      → Deploy permissions only
```

### Credential Types:
- **Username/Password**: For basic auth
- **SSH Key**: For Git, server connections
- **Secret text**: API tokens
- **Secret file**: Certificates, config files
- **Docker Hub credentials**: For registry access

---

## Freestyle vs Pipeline Jobs

| Feature | Freestyle | Pipeline |
|---------|-----------|----------|
| Configuration | GUI-based | Code (Jenkinsfile) |
| Version control | ❌ | ✅ (in Git) |
| Complexity | Simple tasks | Complex workflows |
| Reusability | Low | High |
| Recommended | Legacy | ✅ Modern approach |

---

## Backup & Restore

### What to Back Up:
- `$JENKINS_HOME` directory (contains everything)
- Important subdirectories:
  - `jobs/` – Job configurations and build history
  - `plugins/` – Installed plugins
  - `config.xml` – Global configuration
  - `credentials.xml` – Stored credentials

### Backup Methods:
```bash
# Simple backup using tar
tar -czf jenkins-backup.tar.gz /var/jenkins_home

# Using ThinBackup plugin (recommended)
# Manage Jenkins → ThinBackup → Settings → Backup Now
```

---

## Key Takeaways

1. Jenkins uses a **Master/Agent** architecture for distributed builds
2. **Plugins** are the backbone – they extend every aspect of Jenkins
3. **Pipeline jobs** (Jenkinsfile) are the modern approach over Freestyle
4. Security should be configured with **RBAC** and proper credentials
5. Always **back up** `$JENKINS_HOME` regularly
6. Docker is the easiest way to install and run Jenkins
