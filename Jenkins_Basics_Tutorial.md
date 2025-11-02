# Jenkins Basics Tutorial

## What is Jenkins?

Jenkins is an open-source automation server that helps you build, test, and deploy software continuously. It enables **Continuous Integration (CI)** and **Continuous Deployment (CD)** pipelines.

---

## Jenkins Architecture Overview

```
┌─────────────────────────────────────────────────────────────┐
│                     JENKINS MASTER NODE                     │
│  ┌────────────────────────────────────────────────────────┐ │
│  │  Jenkins Server (Orchestrator)                         │ │
│  │  - Manages jobs, pipelines, and schedules             │ │
│  │  - Web UI & REST API                                  │ │
│  │  - Authentication & Authorization                     │ │
│  └────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────┘
                           │
            ┌──────────────┼──────────────┐
            │              │              │
        ┌───▼────┐    ┌───▼────┐    ┌───▼────┐
        │ Agent 1│    │ Agent 2│    │ Agent 3│
        │(Build) │    │ (Test) │    │(Deploy)│
        └────────┘    └────────┘    └────────┘

Agents (Nodes): Execute build jobs distributed across machines
```

---

## Key Concepts

### 1. **Jobs**
A job is a repeatable process that Jenkins orchestrates. Types include:
- **Freestyle Jobs**: Traditional, basic job configuration
- **Pipeline Jobs**: Complex workflows defined in code
- **Multibranch Pipelines**: Automatic job creation per Git branch

### 2. **Builds**
An execution of a job. Each build has:
- Build number (e.g., #123)
- Status (SUCCESS, FAILURE, UNSTABLE)
- Console output logs
- Artifacts (build results, binaries)

### 3. **Pipelines**
A series of connected stages defined in a `Jenkinsfile`:

```
Stage 1: Checkout    ──▶  Stage 2: Build    ──▶  Stage 3: Test    ──▶  Stage 4: Deploy
   (Get Code)            (Compile)            (Run Tests)           (Release)
```

### 4. **Triggers**
Events that automatically start a build:
- **Poll SCM**: Periodically check version control
- **Webhook**: Triggered by Git push events
- **Scheduled**: Cron-based timing (e.g., nightly builds)
- **Manual**: User-triggered via UI

---

## Basic Job Workflow

```
┌─────────────────┐
│  Trigger Event  │
│ (Git push/time) │
└────────┬────────┘
         │
         ▼
┌─────────────────────┐
│  Jenkins Receives   │
│  Trigger Signal     │
└────────┬────────────┘
         │
         ▼
┌─────────────────────┐
│  Queue Job          │
│  (Wait for agent)   │
└────────┬────────────┘
         │
         ▼
┌─────────────────────┐
│  Assign to Agent    │
│  (Execute build)    │
└────────┬────────────┘
         │
         ▼
┌─────────────────────┐
│  Run Build Steps:   │
│  1. Checkout       │
│  2. Build          │
│  3. Test           │
│  4. Archive        │
└────────┬────────────┘
         │
         ▼
┌─────────────────────┐
│  Build Complete     │
│  (Success/Failure)  │
└────────┬────────────┘
         │
         ▼
┌─────────────────────┐
│  Send Notification  │
│  (Email/Slack)      │
└─────────────────────┘
```

---

## Simple Pipeline Example

### Declarative Pipeline (Recommended)

```groovy
pipeline {
    agent any
    
    stages {
        stage('Checkout') {
            steps {
                echo 'Checking out code...'
                checkout scm
            }
        }
        
        stage('Build') {
            steps {
                echo 'Building application...'
                sh 'npm install'
                sh 'npm run build'
            }
        }
        
        stage('Test') {
            steps {
                echo 'Running tests...'
                sh 'npm test'
            }
        }
        
        stage('Deploy') {
            steps {
                echo 'Deploying to production...'
                sh './deploy.sh'
            }
        }
    }
    
    post {
        always {
            echo 'Cleanup and notifications'
        }
        success {
            echo 'Build succeeded!'
        }
        failure {
            echo 'Build failed!'
        }
    }
}
```

---

## CI/CD Pipeline with Jenkins

```
┌──────────────┐
│  Developer   │
│  Commits     │
└───────┬──────┘
        │
        ▼
┌──────────────────────┐
│  Git Repository      │
│  (GitHub/GitLab)     │
└───────┬──────────────┘
        │ (Webhook)
        ▼
┌──────────────────────┐
│  Jenkins Master      │
│  Triggers Pipeline   │
└───────┬──────────────┘
        │
        ▼
    ┌───┴───┐
    ▼       ▼
 ┌────┐ ┌────────┐
 │ CI │ │Agent 1 │  Checkout ──▶ Build ──▶ Unit Tests ──▶ Code Quality
 └────┘ └────────┘
    
    Pass? ───────▶ Yes
              │
              ▼
          ┌────────┐
          │Agent 2 │  Integration Tests
          └────┬───┘
               │
          Pass? Yes
               │
               ▼
           ┌──────────┐
           │  Staging │  (CD Phase)
           │Deployment│
           └────┬─────┘
                │
           Pass? Yes
                │
                ▼
           ┌──────────┐
           │Production│
           │Deployment│
           └──────────┘
```

---

## Common Jenkins Plugins

| Plugin | Purpose |
|--------|---------|
| **Pipeline** | Define complex workflows as code |
| **Git** | Integrate with Git repositories |
| **GitHub/GitLab** | Webhook integration with hosting platforms |
| **Docker** | Build and push Docker images |
| **Email** | Send email notifications |
| **Slack** | Send notifications to Slack channels |
| **SonarQube** | Code quality analysis |
| **Kubernetes** | Deploy to Kubernetes clusters |
| **JUnit** | Parse and report test results |

---

## Setting Up a Basic Job

### Step 1: Create New Job
```
Dashboard ──▶ New Item ──▶ Enter job name ──▶ Select job type
```

### Step 2: Configure Source Control
```
SCM (Source Code Management):
├─ Select Git
├─ Repository URL: https://github.com/user/repo.git
├─ Credentials: Add your Git credentials
└─ Branches to build: */main
```

### Step 3: Build Triggers
```
Build Triggers:
├─ ☑ GitHub hook trigger for GITScm polling
├─ ☑ Poll SCM
│   └─ Schedule: H/15 * * * *  (Every 15 minutes)
└─ ☑ Build periodically
    └─ Schedule: 0 2 * * *  (Daily at 2 AM)
```

### Step 4: Build Steps
```
Add Build Step:
├─ Execute Shell (Linux/Mac)
│   └─ Command: npm install && npm run build
├─ Execute Windows batch command (Windows)
│   └─ Command: npm install && npm run build
└─ Invoke top-level Maven targets
    └─ Goals: clean package
```

### Step 5: Post-Build Actions
```
Add Post-build Action:
├─ Archive artifacts
│   └─ Files to archive: build/**/*.jar
├─ Send build notifications
│   └─ Slack/Email configuration
└─ Trigger parameterized build on other projects
    └─ Downstream projects: deploy-job
```

---

## Jenkinsfile Best Practices

```groovy
pipeline {
    agent {
        docker {
            image 'node:16'
            args '-v /var/run/docker.sock:/var/run/docker.sock'
        }
    }
    
    options {
        // Keep last 10 builds
        buildDiscarder(logRotator(numToKeepStr: '10'))
        // Timeout after 1 hour
        timeout(time: 1, unit: 'HOURS')
        // Disable concurrent builds
        disableConcurrentBuilds()
    }
    
    environment {
        // Define environment variables
        APP_ENV = 'production'
        BUILD_PATH = 'build'
    }
    
    stages {
        stage('Build') {
            when {
                // Conditional execution
                branch 'main'
            }
            steps {
                script {
                    try {
                        sh 'npm run build'
                    } catch (Exception e) {
                        echo "Build failed: ${e.message}"
                        throw e
                    }
                }
            }
        }
    }
    
    post {
        always {
            // Clean up workspace
            cleanWs()
        }
        success {
            archiveArtifacts artifacts: 'build/**', allowEmptyArchive: false
        }
        failure {
            echo "Notifying team of failure"
        }
    }
}
```

---

## Troubleshooting Common Issues

| Issue | Solution |
|-------|----------|
| **Build never starts** | Check triggers are enabled; verify SCM credentials |
| **Agent not available** | Ensure agent is online; check network connectivity |
| **Out of disk space** | Configure build discarder; clean old builds |
| **Slow builds** | Add parallel stages; use build caching; optimize dependencies |
| **Failed tests ignored** | Set proper test result parsing in post-build actions |

---

## Next Steps

1. **Install Jenkins**: Docker, WAR file, or native installer
2. **Configure Global Settings**: System configuration, plugins
3. **Set Up Credentials**: Git, Docker, cloud providers
4. **Create Your First Job**: Start with a simple build job
5. **Advance to Pipelines**: Write Jenkinsfile for complex workflows
6. **Integrate with Tools**: Docker, Kubernetes, cloud platforms
7. **Monitor & Optimize**: Use metrics and dashboards

---

## Resources

- Official Jenkins Documentation: https://www.jenkins.io/doc/
- Jenkins Pipeline Syntax: https://www.jenkins.io/doc/book/pipeline/
- Jenkins Plugin Repository: https://plugins.jenkins.io/
- Jenkins Community Chat: https://www.jenkins.io/chat/

