
# SecOps Pipeline Template

This repository provides a **reusable GitHub Actions DevSecOps pipeline** for multi-language projects.
It supports:

* SBOM/SCA generation
* SonarQube analysis (**project key must be the repo name**)
* Multi-language linting & auto-fix (Python, Java, JavaScript, PHP, HTML, CSS)
* Docker build & push
* Artifact collection for reports and logs
* Feature toggles for consumer repositories

---

## 🔹 Features

| Feature                    | Description                                                                                                 |
| -------------------------- | ----------------------------------------------------------------------------------------------------------- |
| SBOM / SCA                 | Generates a software bill of materials using codenotary/grypefs.                                            |
| SonarQube Analysis         | Runs SonarQube scans. **Project key must match your repository name** and is automatically derived from it. |
| Multi-Language Lint/Format | Auto-fixes code in Python, Java, JavaScript, PHP, HTML, CSS.                                                |
| Docker Build & Push        | Builds Docker image after lint/format and pushes to DockerHub.                                              |
| Artifact Collection        | Saves SBOM, SonarQube, and linter logs for review.                                                          |
| Workflow Switches          | Consumers can enable/disable SBOM, SonarQube, Docker build.                                                 |
| Language Selection         | Consumers can specify which languages to lint/format.                                                       |

---

## 🔹 How to Use in a Consumer Repo

1. **Create workflow file** in your repository:

```yaml
# .github/workflows/devsecops.yml
name: DevSecOps Pipeline

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
      enable_sast: true
      enable_docker_build: true
      enable_container_scan: true
      languages: "python,javascript,java,php"
    secrets: inherit
```

2. **Secrets Required** (in repository settings → Secrets):

* `SONAR_TOKEN`
* `SONAR_HOST_URL`
* `DOCKERHUB_USERNAME`
* `DOCKERHUB_TOKEN`

> **Remark:** The SonarQube project key **must exactly match your repository name**. The workflow automatically converts the repo name into a valid project key for SonarQube.

---

## 🔹 Inputs / Feature Toggles

| Input                 | Type    | Default                                 | Description                                        |
| --------------------- | ------- | --------------------------------------- | -------------------------------------------------- |
| `enable_sca`          | boolean | `true`                                  | Enable SBOM / SCA scan                             |
| `enable_sonarqube`    | boolean | `true`                                  | Enable SonarQube analysis                          |
| `enable_docker_build` | boolean | `true`                                  | Enable Docker build & push                         |
| `languages`           | string  | `"python,javascript,java,php,html,css"` | Comma-separated list of languages to lint/auto-fix |

---

## 🔹 Supported Languages

* **Python** → `black` + `autopep8`
* **JavaScript** → `prettier` + `eslint`
* **Java** → `google-java-format`
* **PHP** → `phpcbf`
* **HTML/CSS** → `prettier` + `stylelint`

Artifacts for each language are saved in `artifacts/` and uploaded after the workflow.

---

## 🔹 Artifacts Generated

* `sbom-report/` → SBOM URL file
* `sonar-reports/` → SonarQube logs
* `lint-reports/` → Linter logs for each language

All artifacts are available in the **GitHub Actions → Artifacts** tab.

---

## 🔹 Docker Build

Docker images are tagged automatically:

```text
<DOCKERHUB_USERNAME>/${{ github.repository }}:<GITHUB_SHA>
<DOCKERHUB_USERNAME>/${{ github.repository }}:latest
```

Build happens **after all lint/format fixes** have been applied.

---

## 🔹 SonarQube Scan Workflow

The workflow **automatically derives the project key from the repository name** using the following logic:

```yaml
- name: Set Project Key
  run: |
    REPO_NAME=$(echo "${GITHUB_REPOSITORY##*/}" | tr '[:upper:]' '[:lower:]' | sed 's/[^a-z0-9._-]/-/g')
    echo "SONAR_PROJECT_KEY=$REPO_NAME" >> $GITHUB_ENV

- name: SonarQube Scan
  uses: sonarsource/sonarqube-scan-action@v2
  with:
    projectBaseDir: .
    args: >
      -Dsonar.projectKey=${{ env.SONAR_PROJECT_KEY }}
  env:
    SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
    SONAR_HOST_URL: ${{ secrets.SONAR_HOST_URL }}
```

> **Note:** This ensures that the SonarQube project key always matches the repository name and is valid even if the repo name contains uppercase letters or special characters.

