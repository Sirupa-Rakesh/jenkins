# 31-cicd-tools

Terraform configuration to provision a complete **CI/CD infrastructure** for the **Roboshop** project on AWS.

---

# Architecture

```
                                            GitHub
                                                │
                                                ▼
                                        Jenkins Server
                                                │
                                    ┌───────────┴───────────┐
                                    ▼                       ▼
                            Jenkins Agent            SonarQube
                                    │                       │
                                    └───────────┬───────────┘
                                                ▼
                                        Quality Reports
                            ```

---

# Infrastructure Created

## EC2 Instances

| Resource | Instance Type | AMI | Purpose |
|----------|---------------|-----|---------|
| Jenkins Server | t3.small | Redhat-9-DevOps-Practice | Jenkins controller installed using `jenkins.sh` |
| Jenkins Agent | t3.micro | Redhat-9-DevOps-Practice | Jenkins build agent with Java installed using `jenkins-agent.sh` |
| SonarQube Server | t3.large | SolveDevOps-SonarQube-Server-Ubuntu24.04 | SonarQube server (created only when `var.sonar=true`) |

### Storage

| Instance | Root Volume |
|-----------|-------------|
| Jenkins Server | Default |
| Jenkins Agent | 50 GB gp3 |
| SonarQube | 20 GB gp3 |

---

# Route53 DNS Records

| DNS Record | Type | Points To |
|------------|------|-----------|
| `jenkins.<domain>` | A | Jenkins Server Public IP |
| `jenkins-agent.<domain>` | A | Jenkins Agent Private IP |
| `sonar.<domain>` | A | SonarQube Public IP (only when Sonar enabled) |

Default Domain

```
rakeshdev.online
```

---

# SSM Parameters Used

Terraform reads these values during apply.

| Parameter | Description |
|-----------|-------------|
| `/<project>/<env>/public_subnet_ids` | Public subnet ID |
| `/<project>/<env>/jenkins_sg_id` | Jenkins Security Group |
| `/<project>/<env>/jenkins_agent_sg_id` | Jenkins Agent Security Group |
| `/<project>/<env>/sonar_sg_id` | SonarQube Security Group |

---

# Variables

| Variable | Default | Description |
|----------|---------|-------------|
| project | roboshop | Project Name |
| environment | dev | Environment |
| zone_id | Z05013202FKF0ZL12WAOP | Route53 Hosted Zone ID |
| domain_name | daws88s.online | Base Domain |
| sonar | true | Enable or Disable SonarQube |

---

# Deploy Without SonarQube

```bash
terraform apply -var="sonar=false"
```

---

# Jenkins Configuration

After opening Jenkins for the first time:

```
http://jenkins.<domain>:8080
```

Complete the setup wizard and install the required plugins.

---

# Required Plugins

- Pipeline Stage View
- Pipeline Utility Steps
- AWS Credentials
- AWS Steps
- SonarQube Scanner
- Multibranch Scan Webhook Trigger
- JIRA Pipeline Steps
- Generic Webhook Trigger

---

# Jenkins Credentials

Create the following credentials.

| Credential ID | Purpose |
|--------------|---------|
| ssh-creds | SSH Authentication |
| aws-creds | AWS Access |
| sonar-creds | Sonar Authentication Token |
| github-token | GitHub Personal Access Token |
| jira-creds | JIRA API Credentials |

---

# GitHub Fine-Grained Token

Navigate to

```
Profile
    ↓
Settings
    ↓
Developer Settings
    ↓
Fine-grained Personal Access Token
```

Select

```
All Repositories
```

Required Permissions

| Permission | Access |
|------------|--------|
| Code | Read & Write |
| Commit Statuses | Read & Write |
| Dependabot Alerts | Read |

---

# Jenkins Master Node

Configure

```
Agent Name:
jenkins-agent.daws88s.online
```

Label

```
roboshop
```

---

# SonarQube Configuration

## Scanner Tool

```
Manage Jenkins

↓

Global Tool Configuration

↓

Sonar Scanner

↓

Name

sonar-8
```

---

## Sonar Server

```
Manage Jenkins

↓

System

↓

SonarQube Servers

↓

Name

sonar-server
```

Add

- Server URL
- Authentication Token

---

# SonarQube Webhook

```
Administration

↓

Configuration

↓

Webhooks

↓

Create Webhook
```

Mode

```
Standard
```

---

# Configure Quality Gate

Create a Quality Gate inside SonarQube and use it in Jenkins Pipeline.

---

# Jenkins Shared Library

Configure Global Trusted Library.

```
Manage Jenkins

↓

System

↓

Global Trusted Pipeline Libraries
```

Repository

```
jenkins-shared-library
```

---

# SonarQube Metrics

## 🐞 Bugs

A **Bug** is a coding mistake that may cause unexpected application behavior or failure.

Example

- Null Pointer Exception
- Array Index Out of Bounds
- Division by Zero

**Impact**

Application stability.

---

## 🔐 Vulnerabilities

A **Vulnerability** is a security weakness that attackers can exploit.

Examples

- SQL Injection
- Cross-Site Scripting (XSS)
- Hardcoded Passwords

**Impact**

Security of the application.

---

## 👃 Code Smells

A **Code Smell** is not a bug.

The application works correctly, but the code is difficult to understand, maintain, or extend.

Examples

- Long Methods
- Duplicate Code
- Unused Variables
- Large Classes
- Poor Naming

**Impact**

Maintainability.

---

# Technical Debt

Technical Debt is the estimated time required to fix all maintainability issues.

Example

```
100 Code Smells

Each requires

10 minutes

Technical Debt

100 × 10

=

1000 minutes

≈16.6 hours
```

Typical SonarQube Estimates

| Issue | Estimated Fix Time |
|--------|-------------------|
| Rename Variable | 2 minutes |
| Remove Duplication | 1 hour |
| Reduce Complexity | 30 minutes |

---

# 📋 Duplication

Duplication measures repeated or nearly identical code.

Example

```
File A

login()

File B

login()

Same code copied twice.
```

Better approach

Create one reusable function.

---

# 📊 Coverage

Coverage measures how much source code is executed during unit testing.

Formula

```
Coverage

=

(Lines Covered / Lines to Cover)

×

100
```

Example

```
Total Lines

1000

Covered

850

Coverage

85%
```

Unit testing generates reports such as

- Total Test Cases
- Passed
- Failed
- Coverage

The Jenkins Agent uploads these reports to SonarQube.

---

# 🔒 Security Rating

Security Rating depends on the highest severity vulnerability.

| Rating | Meaning |
|---------|---------|
| A | No Vulnerabilities |
| B | Minor Issues |
| C | Moderate Issues |
| D | Major Issues |
| E | Critical Vulnerability |

Example

```
1 Critical Vulnerability

↓

Security Rating

E
```

---

# 🛠️ Maintainability Rating

Maintainability Rating is calculated using the Technical Debt Ratio.

Formula

```
Technical Debt Ratio

=

(Remediation Cost / Development Cost)

×

100
```

---

## Total Remediation Cost

Time required to fix all Code Smells.

Example

| Issue | Time |
|-------|------|
| Duplicate Code | 1 Hour |
| Long Method | 30 Minutes |
| Variable Rename | 2 Minutes |

Total

```
1 Hour 32 Minutes
```

---

## Development Cost

SonarQube estimates the development effort as:

```
Development Cost

=

Lines of Code

×

30 Minutes
```

Example

```
1000 Lines

↓

1000 × 30

↓

30000 Minutes

↓

500 Hours
```

---

## Example

```
Lines of Code

2000

Development Cost

1000 Hours

Remediation Cost

50 Hours

Technical Debt Ratio

50 / 1000

=

5%
```

---

## Maintainability Ratings

| Ratio | Rating |
|-------|--------|
| ≤5% | A |
| 6–10% | B |
| 11–20% | C |
| 21–50% | D |
| >50% | E |

Factors affecting Maintainability

- Code Smells
- Complexity
- Duplication

Maintainability indicates how easy it is to modify and maintain the application in the future.

---

# 🎯 Reliability Rating

Reliability measures application stability.

It is based on the severity of Bugs.

| Rating | Meaning |
|---------|---------|
| A | No Bugs |
| B | Minor Bugs |
| C | Moderate Bugs |
| D | Major Bugs |
| E | Critical Bugs |

Critical or Blocker bugs reduce the Reliability Rating significantly.

---

# CI/CD Workflow

```
Developer

↓

GitHub

↓

Webhook

↓

Jenkins

↓

Checkout Code

↓

Build

↓

Unit Testing

↓

Generate Coverage Report

↓

SonarQube Scan

↓

Quality Gate

↓

Docker Build

↓

Push Image to Amazon ECR

↓

Deploy to Kubernetes / EKS
```

---

# Project Structure

```
31-cicd-tools
│
├── main.tf
├── variables.tf
├── outputs.tf
├── datasource.tf
├── route53.tf
├── ec2.tf
├── jenkins.sh
├── jenkins-agent.sh
├── sonarqube.sh
└── README.md
```

---

# Cleanup

Destroy all resources:

```bash
terraform destroy
```

Destroy without SonarQube:

```bash
terraform destroy -var="sonar=false"
```

---

# Author

**Rakesh Sirupa**

DevOps | AWS | Terraform | Jenkins | Docker | Kubernetes | SonarQube | CI/CD