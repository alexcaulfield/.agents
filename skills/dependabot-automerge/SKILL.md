---
name: dependabot-automerge
description: Automates the setup of Dependabot updates, CI checks, and auto-merge workflow configurations for GitHub repositories.
---

# Dependabot Automerge Setup Skill

## Role
You are a Senior DevOps and Platform Engineer helping the user automate dependency updates and pull request merging. Your goal is to configure Dependabot, establish CI status verification, and set up an automatic PR merging workflow so that dependencies are kept up-to-date with zero manual intervention once tests pass.

## When to use this skill
- Use this when the user asks to configure, initialize, or setup Dependabot in a repository.
- Use this when the user wants to enable auto-merging for Dependabot pull requests.
- Use this when bootstrapping or updating repository DevOps/CI configurations.

## How to use it

Perform the following steps sequentially to configure the repository:

### 1. Detect Package Ecosystems
Inspect the workspace to determine which package managers and language runtimes are used, and identify their directories (e.g., frontend root, backend subdirectory, serverless folders, dockerfiles).
Common ecosystems include:
- `npm` (Node.js/TypeScript packages)
- `pip` (Python dependencies)
- `cargo` (Rust crates)
- `docker` (Docker files)
- `github-actions` (GitHub Actions workflow versions)

### 2. Configure Dependabot
Create or update `.github/dependabot.yml`.
Ensure it has:
- `version: 2`
- Updates mapped to the discovered ecosystems.
- A scheduled update interval (e.g., `weekly` or `monthly`).
- Appropriate labels (e.g., `dependencies`) and default assignees (e.g., `alexcaulfield`).

Example:
```yaml
version: 2
updates:
  - package-ecosystem: "npm"
    directory: "/"
    schedule:
      interval: "monthly"
    labels:
      - "dependencies"
    assignees:
      - "alexcaulfield"
```

### 3. Verify or Create CI Workflow
Check if there is an existing CI test/build workflow running on pull requests (e.g. `.github/workflows/ci.yml`). If not, scaffold a basic CI workflow tailored to the project stack to ensure status checks are run. This is necessary because auto-merge requires passing status checks to verify safety before merging.

### 4. Create Auto-Merge Workflow
Create `.github/workflows/dependabot-automerge.yml` with contents to automatically enable auto-merging for Dependabot pull requests once checks pass.

Template:
```yaml
name: Dependabot automerge

on: pull_request

permissions:
  contents: write
  pull-requests: write

jobs:
  automerge:
    runs-on: ubuntu-latest
    if: github.actor == 'dependabot[bot]'
    steps:
      - name: Enable automerge for Dependabot PRs
        run: gh pr merge --auto --squash "$PR_URL"
        env:
          PR_URL: ${{ github.event.pull_request.html_url }}
          GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

### 5. Instruct on GitHub Repository Settings
Inform the user that they must verify/enable the following settings on GitHub for the automated workflows to succeed:
1. **Enable Auto-Merge**: Go to `Settings -> General` on GitHub, scroll to the Pull Requests section, and verify **Allow auto-merge** is checked.
2. **Workflow Permissions**: Go to `Settings -> Actions -> General`, scroll to Workflow permissions, and verify **Allow GitHub Actions to create and approve pull requests** is checked.
3. **Branch Protection / Rulesets**: Ensure that status checks (from step 3) are set as required for the default branch (`main` or `master`) so that PRs wait for builds to pass before merging.
