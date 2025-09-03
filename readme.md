# SecOps Pipeline Template

This repository provides a **reusable GitHub Actions DevSecOps pipeline** for multi-language projects.
It supports:

* SBOM/SCA generation
* SonarQube analysis (project key automatically uses repo name)
* Multi-language linting & auto-fix (Python, Java, JavaScript, PHP, HTML, CSS)
* Docker build & push
* Artifact collection for reports and logs
* Feature toggles for consumer repositories

---

## 🔹 Features

| Feature                    | Description                                                                                  |
| -------------------------- | -------------------------------------------------------------------------------------------- |
| SBOM / SCA                 | Generates a software bill of materials using codenotary/grypefs.                             |
| SonarQube Analysis         | Runs SonarQube scans with project key derived from repo name and collects logs as artifacts. |
| Multi-Language Lint/Format | Auto-fixes code in Python, Java, JavaScript, PHP, HTML, CSS.                                 |
| Docker Build & Push        | Builds Docker image after lint/format and pushes to DockerHub.                               |
| Artifact Collection        | Saves SBOM, SonarQube, and linter logs for review.                                           |
| Workflow Switches          | Consumers can enable/disable SBOM, SonarQube, Docker build.                                  |
| Language Selection         | Consumers can specify which languages to lint/format.                                        |

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
      enable_sonarqube: true
      enable_docker_build: true
      languages: "python,javascript,java,php"
    secrets: inherit
```

2. **Secrets Required** (in repository settings → Secrets):

* `SONAR_TOKEN`
* `SONAR_HOST_URL`
* `DOCKERHUB_USERNAME`
* `DOCKERHUB_TOKEN`

> **Note:** `SONAR_PROJECT_KEY` is **no longer required**. The workflow automatically converts your repo name to a valid SonarQube project key.

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

