# REPO_SUMMARY.md

## Overview

This is a **GitHub Actions learning repository** (`alice-app`) containing a minimal Node.js app used as a sandbox to experiment with GitHub Actions features. The repository's primary purpose is education: each workflow file demonstrates a distinct GitHub Actions concept, from basic triggers and job ordering to Docker containers, matrix builds, and workflow commands.

---

## Project Structure

```
github_action/
├── .github/
│   └── workflows/          # 16 workflow files (one concept each)
│       ├── main.yml
│       ├── action.yml
│       ├── steps.yml
│       ├── jobs.yml
│       ├── push-pr.yml
│       ├── scheduled.yml
│       ├── manual.yml
│       ├── api-manual.yml
│       ├── command.yml
│       ├── comman-2.yml
│       ├── command-3.yml
│       ├── command-4.yml
│       ├── docker.yml
│       ├── expression.yml
│       ├── expression-2.yml
│       └── metrix.yml
├── src/
│   ├── app.js              # Tiny Node.js script (3 variables + console.log)
│   └── test.sh             # Empty shell script placeholder
├── package.json            # npm project manifest
└── README.md               # Minimal title only
```

---

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Runtime | Node.js (v14/15/16 tested in matrix) |
| Package manager | npm |
| CI/CD | GitHub Actions |
| Containers | Docker (Alpine 3.14, node:20) |
| Scripting | Bash, Perl (used in steps.yml demo) |

The app itself (`src/app.js`) is intentionally trivial — it just prints three variables. The real content of this repo lives in the workflow files.

---

## GitHub Actions / CI-CD Explained

Each workflow file teaches one specific feature. Here they are in logical order:

---

### 1. `main.yml` — Basic push trigger
- **Trigger:** Push to `main` branch
- **Jobs:** One job (`example-jop`) on `ubuntu-latest`
- **Steps:** Print a welcome message → checkout the repo → list files with `ls -la`
- **Purpose:** The "Hello World" of GitHub Actions. Shows the simplest possible workflow.

---

### 2. `push-pr.yml` — Push + Pull Request with path filters
- **Trigger:** Push or pull_request to `main` or any `release/**` branch, **but only when files inside `src/` change**
- **Jobs:** One job (`test`) — checks out code and runs `npm test`
- **Purpose:** Shows how to avoid wasted runs by filtering on branch names and file paths. The workflow only fires if source code actually changed.

---

### 3. `scheduled.yml` — Cron schedule
- **Trigger:** Cron schedule — `0 * * * *` (top of every hour)
- **Jobs:** One job (`daily-task`) that prints "Running the schedule"
- **Purpose:** Demonstrates scheduled/automated workflows, useful for nightly builds, cleanup jobs, or health checks.

---

### 4. `manual.yml` — Manual trigger with input
- **Trigger:** `workflow_dispatch` (manual button in GitHub UI)
- **Inputs:** `environment` — a dropdown choice between `staging` and `production` (default: `staging`)
- **Jobs:** One job (`deploy`) that echoes which environment was chosen
- **Purpose:** Shows how to build manually-triggered deployment workflows with user-provided parameters.

---

### 5. `api-manual.yml` — External API trigger
- **Trigger:** `repository_dispatch` with event type `incident_report`
- **Jobs:** One job (`handle-incident`) that prints "incident report"
- **Purpose:** Shows how an external system (a script, webhook, or another service) can kick off a workflow via the GitHub API by posting a custom event.

---

### 6. `action.yml` — Using a local custom action
- **Trigger:** `workflow_dispatch`
- **Jobs:** One job (`actions_job`) that calls a local custom action at `.github/action/Greeting@v1`
- **Secrets used:** `GITHUB_TOKEN` (auto-provided by GitHub)
- **Inputs passed:** `name: ahmed`
- **Purpose:** Demonstrates how to write and consume reusable local actions within the same repo.

---

### 7. `steps.yml` — Step options (working directory, shell)
- **Trigger:** `workflow_dispatch`
- **Jobs:** One job (`job_1`) with two steps:
  - Runs `npm install && npm run build` inside a `./temp` working directory using the `bash` shell
  - Runs `print %ENV` using the `perl` shell (`perl {0}`)
- **Purpose:** Shows that steps can override the working directory and the shell used (bash, PowerShell, Python, cmd, perl, etc.).

---

### 8. `jobs.yml` — Job dependencies (`needs`)
- **Trigger:** `workflow_dispatch`
- **Jobs:** Four jobs demonstrating a dependency chain:
  - `job_1` runs first (no dependencies)
  - `job_2` and `job_3` both wait for `job_1` (run in parallel after it)
  - `job_4` waits for both `job_2` and `job_3`
- **Purpose:** Shows how to sequence jobs into a DAG (directed acyclic graph) using the `needs` keyword.

---

### 9. `command.yml` — Workflow commands: warnings
- **Trigger:** `workflow_dispatch`
- **Steps:** Sets a bash variable and uses `echo "::warning::..."` to emit a warning annotation in the GitHub UI if a deprecated feature is detected.
- **Purpose:** Introduces workflow commands — special `echo` patterns that communicate with the runner (set warnings, errors, debug messages, masks, environment variables, etc.).

---

### 10. `comman-2.yml` — Workflow commands: environment variables
- **Trigger:** `workflow_dispatch`
- **Steps:** Writes `API_KEY=12345` to `$GITHUB_ENV` so subsequent steps can read it as an environment variable.
- **Purpose:** Shows the correct way to pass a value from one step to later steps using `$GITHUB_ENV`.

---

### 11. `command-3.yml` — Workflow commands: debug and error
- **Trigger:** `workflow_dispatch`
- **Steps:** Checks whether `config.yml` exists and emits `::debug::` or `::error::` annotations accordingly.
- **Purpose:** Demonstrates using workflow commands to surface debug info and errors as visible annotations in the Actions UI.

---

### 12. `command-4.yml` — Workflow commands: masking secrets
- **Trigger:** `workflow_dispatch`
- **Steps:**
  1. Sets `API_KEY=12345` (note: this version has a bug — it writes to `GITHUB_ENV` without the `$` prefix)
  2. Uses `echo "::add-mask::$API_KEY"` to mask the value in logs
- **Purpose:** Shows `::add-mask::` which redacts a value from all future log output — important for protecting sensitive data.

---

### 13. `docker.yml` — Docker containers in jobs
- **Trigger:** `workflow_dispatch`
- **Jobs:** Two jobs:
  - `hybrid_job`: Mixes VM steps and a Docker container step (`docker://alpine:3.14`) within the same job
  - `dockerized_job`: Runs the **entire job** inside a `node:20` container (pulled from Docker Hub using secrets `DOCKER_USERNAME` / `DOCKER_PASSWORD`)
- **Secrets used:** `DOCKER_USERNAME`, `DOCKER_PASSWORD`
- **Purpose:** Shows two ways to use containers — per-step (`uses: docker://image`) and per-job (`container:` key).

---

### 14. `expression.yml` — Conditional step (`if`)
- **Trigger:** `workflow_dispatch`
- **Steps:** Runs `echo "The job is running"` only if `github.event_name == 'push'` OR the ref is `refs/head/main`
- **Purpose:** Demonstrates `if:` conditions using GitHub context expressions to make steps conditional.

---

### 15. `expression-2.yml` — Status check functions
- **Trigger:** `workflow_dispatch`
- **Steps:** Runs a deploy script, then runs a cleanup step only `if: failure()` (i.e., only when a previous step failed)
- **Purpose:** Shows status functions like `failure()`, `success()`, `always()` for conditional cleanup or notification steps.

---

### 16. `metrix.yml` — Build matrix strategy
- **Trigger:** `workflow_dispatch`
- **Jobs:** One job (`job_1`) with a matrix strategy:
  - OS: `ubuntu-latest`, `windows-latest`, `macos-latest`
  - Node versions: `14.x`, `15.x`, `16.x`
  - Produces **9 parallel jobs** (3 OS × 3 Node versions)
  - `fail-fast: false` — one failed combo won't cancel the rest
  - `max-parallel: 3` — at most 3 run at the same time
- **External action:** `actions/setup-node@v3.6.0`
- **Purpose:** Shows matrix builds for cross-platform / multi-version testing.

---

## How the CI/CD Pipeline Fits Together

This repo does **not** have a single linear pipeline. Instead, each workflow is an independent teaching example. Here's how they map to real-world CI/CD stages:

```
Trigger Type          Workflow File        Real-World Equivalent
─────────────────     ─────────────────    ─────────────────────────
Push to main          main.yml             Smoke test on every commit
Push to src/          push-pr.yml          Run tests only when code changes
Cron (hourly)         scheduled.yml        Scheduled health check / nightly build
Manual (UI)           manual.yml           Manual deploy to staging or production
External API call     api-manual.yml       Incident response / ChatOps trigger
Matrix (9 combos)     metrix.yml           Cross-platform compatibility testing
Docker container      docker.yml           Containerized build / isolated environment
```

The workflows that use `workflow_dispatch` are standalone demos — they have no automatic trigger and must be run manually from the GitHub Actions tab.

---

## Key Things to Know for a Newcomer

1. **This is a learning repo, not a production app.** The Node.js app is just a placeholder. Everything interesting is in `.github/workflows/`.

2. **Each workflow file = one concept.** Read them in the order listed above (main → push-pr → scheduled → manual → ...) for a logical progression from simple to advanced.

3. **Workflow triggers decide when a workflow runs:**
   - `push` / `pull_request` — reacts to Git events
   - `schedule` — runs on a cron timer
   - `workflow_dispatch` — manual button in the GitHub UI
   - `repository_dispatch` — triggered by an external API call

4. **`needs:` creates job ordering.** Without it, all jobs run in parallel. Use `needs: [job_a, job_b]` to make a job wait for others.

5. **`$GITHUB_ENV` is how you share variables between steps** inside the same job. Write `KEY=VALUE >> $GITHUB_ENV`, then read `$KEY` in later steps.

6. **Workflow commands** (`echo "::warning::"`, `echo "::error::"`, `echo "::add-mask::"`) are special strings that communicate with the GitHub runner to surface annotations, mask secrets, or set outputs.

7. **Secrets to configure** if you fork this repo and run the Docker workflow:
   - `DOCKER_USERNAME` and `DOCKER_PASSWORD` — needed by `docker.yml`
   - `GITHUB_TOKEN` — automatically available, used in `action.yml`

8. **Known issue in `command-4.yml`:** The first step writes to `GITHUB_ENV` without the `$` prefix, so the variable won't actually be exported. This is a demo bug — correct usage is `echo "KEY=value" >> $GITHUB_ENV`.

9. **`metrix.yml` naming** is a typo for "matrix" — this is the build matrix demo showing cross-platform testing.
