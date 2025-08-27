# Security Pipeline Documentation

This document describes the comprehensive DevSecOps pipeline implemented for this project.

## Pipeline Overview

The CI/CD pipeline includes the following security checks and stages:

### Security Jobs (Must Pass Before Build)

1. **Unit Tests** (`unit-test`)
   - Runs pytest on all test files
   - Ensures code functionality before security scanning

2. **SAST - Static Application Security Testing** (`sast`)
   - Uses Bandit to scan Python code for security vulnerabilities
   - Configuration: `.bandit` file
   - Generates JSON and text reports

3. **SCA - Software Composition Analysis** (`sca`)
   - Uses pip-audit to check for known vulnerabilities in dependencies
   - Scans both production and development requirements
   - Fails if any vulnerabilities are found (no `|| true` workarounds)

4. **Secret Scanning** (`secret-scan`)
   - Uses Gitleaks to scan for hardcoded secrets
   - Configuration: `.gitleaks.toml` file
   - Scans entire git history

### Build and Deployment Jobs

5. **Build & Push Docker Image** (`build_and_push`)
   - Only runs if ALL security jobs pass (`needs: [unit-test, sast, sca, secret-scan]`)
   - Builds multi-architecture Docker image (linux/amd64, linux/arm64)
   - Pushes to Docker Hub with tags:
     - `:latest` (for main branch)
     - `:<branch>-<commit-sha>`
     - `:<commit-sha>`
   - Generates SBOM (Software Bill of Materials)

6. **Container Image Scanning** (`image-scan`)
   - Uses Trivy to scan the built Docker image from Docker Hub
   - Uploads results to GitHub Security tab (SARIF format)
   - Fails if HIGH or CRITICAL vulnerabilities are found
   - Generates detailed JSON report

### Summary Job

7. **Security Summary** (`security-summary`)
   - Provides comprehensive status overview
   - Runs regardless of other job outcomes
   - Displays results in GitHub Actions summary

## Required Secrets

Configure these secrets in your GitHub repository:

- `DOCKERHUB_USERNAME`: Your Docker Hub username
- `DOCKERHUB_TOKEN`: Docker Hub access token (not password!)

## Dependency Management

### Updated Dependencies (Bagian B)

The following dependencies have been updated to address security vulnerabilities:

**requirements.txt:**
- `fastapi==0.116.1` (updated from 0.115.0)
- `uvicorn==0.35.0` (updated from 0.30.6)

**dev-requirements.txt:** (no vulnerabilities found)
- `pytest==8.3.2`
- `httpx==0.27.2`
- `bandit==1.7.9`
- `pip-audit==2.7.3`

### Verification

Run the following commands to verify no vulnerabilities exist:

```bash
pip-audit -r requirements.txt
pip-audit -r dev-requirements.txt
```

Both should return "No known vulnerabilities found".

## Configuration Files

- `.github/workflows/ci-cd.yml`: Main CI/CD pipeline
- `.bandit`: Bandit SAST configuration
- `.gitleaks.toml`: Gitleaks secret scanning configuration

## Pipeline Triggers

The pipeline runs on:
- Push to `main`, `develop`, or `sesi30-devsecops` branches
- Pull requests to `main` or `develop` branches

## Security Features

- **Dependency Caching**: Speeds up builds while maintaining security
- **Multi-stage Security**: Each security tool runs independently
- **Fail-Fast**: Build only proceeds if all security checks pass
- **Comprehensive Reporting**: All scan results uploaded as artifacts
- **GitHub Security Integration**: Results visible in GitHub Security tab
- **SBOM Generation**: Software Bill of Materials for supply chain security

## Image Tagging Strategy

Images are tagged with:
- `latest`: Latest stable version (main branch only)
- `<branch>-<commit-sha>`: Branch-specific builds
- `<commit-sha>`: Commit-specific builds for traceability
