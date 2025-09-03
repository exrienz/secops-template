# 🛡️ SecOps Pipeline Template

This repository provides a **reusable DevSecOps GitHub Actions pipeline** that can be plugged into any project.  
It includes **security scans, linting/auto-fix, and Docker build & push** in a consistent way across repositories.

---

## 🚀 Features

- **SCA (Software Composition Analysis)**  
  - SBOM generation with [codenotary/sbom.sh](https://github.com/codenotary/sbom.sh)  

- **SAST (Static Application Security Testing)**  
  - Python security scan with **Bandit**  
  - **SonarQube** scan support  

- **Linting & Auto-Fix (Multi-Language)**  
  - Python → `black`, `autopep8`  
  - JavaScript → `eslint`, `prettier`  
  - Java → `google-java-format`  
  - PHP → `phpcbf`  
  - HTML/CSS → `prettier`, `stylelint`  

- **Docker Build & Push**  
  - Auto-tagged by commit SHA and `latest`  
  - Pushes to DockerHub (secrets required)  

---

## 📦 How to Use in Your Repo

1. Add a workflow file in your project:  
   `.github/workflows/devsecops.yml`

```yaml
name: DevSecOps Pipeline (Consumer)

on:
  push:
    branches: [main, develop]
  pull_request:
  workflow_dispatch:

jobs:
  devsecops:
    uses: exrienz/secops-template/.github/workflows/devsecops.yml@main
    with:
      enable_sca: true
      enable_sonarqube: false
      enable_docker_build: true
      languages: "python,javascript,java,php,html,css" # choose your stack
    secrets: inherit
