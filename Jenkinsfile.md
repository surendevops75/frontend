# Jenkins Shared Library Pipeline Example

This Jenkins Pipeline demonstrates how to use a **Jenkins Shared Library** to centralize and reuse CI/CD logic across multiple projects.

Instead of writing the entire pipeline inside every Jenkinsfile, common pipeline code is stored in a Shared Library and invoked as a function.

This approach helps:

- Reduce code duplication
- Standardize CI/CD processes
- Improve maintainability
- Reuse pipeline logic across multiple repositories

---

# Pipeline Flow

```text
Developer Pushes Code
          │
          ▼
     Jenkinsfile
          │
          ▼
Load Shared Library
          │
          ▼
Check Branch Name
          │
    ┌─────┴─────┐
    ▼           ▼
Feature      Main
Branch       Branch
    │           │
    ▼           ▼
Execute      Skip
Pipeline     Pipeline
Function
```

---

# Jenkinsfile

```groovy
// --------------------------------------------------
// Load Jenkins Shared Library
// --------------------------------------------------

@Library('jenkins-shared-library') _

// --------------------------------------------------
// Application Configuration
// --------------------------------------------------

def configMap = [

    // Project name
    project : "roboshop",

    // Component name
    component: "frontend"
]

// --------------------------------------------------
// Branch Validation
// --------------------------------------------------

// Execute pipeline only for non-main branches

if( ! env.BRANCH_NAME.equalsIgnoreCase('main') ){

    // Call reusable shared library function
    nodeJSEKSPipeline(configMap)

}
else{

    echo "Please proceed with PROD process"
}
```

---

# What is a Jenkins Shared Library?

A Shared Library is a collection of reusable Groovy scripts that can be shared across Jenkins pipelines.

Instead of:

```text
Project A → 500 lines pipeline
Project B → 500 lines pipeline
Project C → 500 lines pipeline
```

You create:

```text
Shared Library
      │
      ├── Build Logic
      ├── Test Logic
      ├── Docker Logic
      ├── Deployment Logic
      └── Security Logic
```

and call it from all projects.

---

# Shared Library Structure

Example:

```text
jenkins-shared-library/
│
├── vars/
│   └── nodeJSEKSPipeline.groovy
│
├── src/
│
└── resources/
```

---

# Library Import

```groovy
@Library('jenkins-shared-library') _
```

Loads the configured Jenkins Shared Library.

The underscore (`_`) is required syntax for loading global libraries.

Purpose:

```text
Import reusable pipeline functions
```

Example:

```groovy
nodeJSEKSPipeline()
javaPipeline()
pythonPipeline()
dockerPipeline()
```

---

# Configuration Map

```groovy
def configMap = [
    project : "roboshop",
    component: "frontend"
]
```

Creates a map containing application configuration.

Equivalent to:

```json
{
  "project": "roboshop",
  "component": "frontend"
}
```

---

# Project Parameter

```groovy
project : "roboshop"
```

Represents the application project.

Used by the shared library for:

```text
Docker Repository Naming
Kubernetes Namespace
Deployment Configuration
Monitoring Labels
```

---

# Component Parameter

```groovy
component : "frontend"
```

Represents the microservice being deployed.

Examples:

```text
frontend
catalogue
cart
user
payment
shipping
```

The shared library can use this value to:

```text
Build Docker Image
Deploy Helm Chart
Create Kubernetes Resources
```

---

# Branch Validation

```groovy
env.BRANCH_NAME
```

Built-in Jenkins environment variable.

Contains:

```text
main
feature-login
feature-payment
bugfix-cart
release-v1
```

Current Git branch.

---

# Case-Insensitive Comparison

```groovy
equalsIgnoreCase('main')
```

Compares branch names without considering case.

Examples:

```text
main      → Match
MAIN      → Match
Main      → Match
```

Purpose:

```text
Prevent case-related issues
```

---

# NOT Operator

```groovy
!
```

Negates the condition.

Meaning:

```groovy
if branch != main
```

Equivalent logic:

```groovy
if(env.BRANCH_NAME != "main")
```

---

# Feature Branch Execution

```groovy
nodeJSEKSPipeline(configMap)
```

Calls a reusable function from the Shared Library.

This function likely performs:

```text
Checkout Source Code
Install Dependencies
Run Unit Tests
Build Docker Image
Security Scan
Push Image
Deploy to Dev Environment
```

All logic is hidden inside:

```text
vars/nodeJSEKSPipeline.groovy
```

---

# Main Branch Behavior

```groovy
echo "Please proceed with PROD process"
```

When code is pushed to:

```text
main
```

the feature pipeline is skipped.

Output:

```text
Please proceed with PROD process
```

Purpose:

```text
Prevent accidental production deployment
```

Production deployments usually require:

- Manual Approval
- Change Management
- Additional Testing
- Security Validation

---

# Example Execution

## Feature Branch

Branch:

```text
feature-login
```

Flow:

```text
Load Shared Library
        │
        ▼
Branch != main
        │
        ▼
Execute nodeJSEKSPipeline()
```

Result:

```text
Build
Test
Docker
Deploy
```

---

## Main Branch

Branch:

```text
main
```

Flow:

```text
Load Shared Library
        │
        ▼
Branch == main
        │
        ▼
Print Message
```

Result:

```text
Please proceed with PROD process
```

---

# Benefits of Shared Libraries

## Reusability

One pipeline implementation can be used by many repositories.

---

## Centralized Maintenance

Update once:

```text
Shared Library
```

Changes apply everywhere.

---

## Consistency

All projects follow:

```text
Same Build Process
Same Testing Standards
Same Deployment Process
```

---

## Reduced Code Duplication

Without Shared Library:

```text
20 Repositories
20 Jenkinsfiles
20 Pipeline Implementations
```

With Shared Library:

```text
20 Repositories
1 Shared Library
```

---

# Real DevOps Use Cases

## Microservices

Repositories:

```text
frontend
catalogue
cart
user
payment
shipping
```

all use the same pipeline.

---

## Standardized CI/CD

Every service follows:

```text
Build
Test
Scan
Deploy
```

---

## Enterprise Platforms

Large organizations use Shared Libraries to enforce:

```text
Security Policies
Deployment Standards
Compliance Requirements
```

---

# Best Practices

✅ Store reusable logic in Shared Libraries

✅ Pass configuration using maps

✅ Separate feature and production workflows

✅ Use branch-based execution

✅ Keep Jenkinsfiles lightweight

✅ Version control shared libraries

---

# Benefits

- Reusable Pipeline Logic
- Easier Maintenance
- Consistent CI/CD
- Reduced Duplication
- Faster Onboarding
- Enterprise-Ready Design

---

# Why This Pipeline Is Important

This Jenkinsfile demonstrates advanced Jenkins concepts:

- Shared Libraries
- Reusable Functions
- Branch-Based Execution
- Configuration Maps
- Centralized CI/CD

These concepts are widely used in:

- DevOps Engineering
- Platform Engineering
- Enterprise CI/CD Platforms
- Cloud-Native Deployments
- Kubernetes-Based Applications