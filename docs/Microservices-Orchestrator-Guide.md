# Microservices-Orchestrator — Usage Guide

This repository is an **orchestrator** for multiple microservices.  
It contains infrastructure (e.g., `docker-compose.yml`, docs) and references each microservice as a **Git submodule**.

## Repositories

Orchestrator:
- `Microservices-Orchestrator` (this repo)

Microservices (submodules):
- Authentication-Service — https://github.com/CountryBoy0ner/Authentication-Service
- User-Service — https://github.com/CountryBoy0ner/User-Service
- Order-Service — https://github.com/CountryBoy0ner/Order-Service
- Payment-Service — https://github.com/CountryBoy0ner/Payment-Service
- API-Gateway — https://github.com/CountryBoy0ner/API-Gateway

---

## How submodules work (quick explanation)

- The orchestrator repo does **not** store the full code of each service.
- Instead, it stores a **pointer** (a specific commit SHA) for each service.
- You edit/commit code **inside** the service repo (submodule).
- Then you commit an updated pointer **in the orchestrator repo** so others get the same version.

---

## Prerequisites

- Git (recent version recommended)
- Docker Desktop (if you plan to run via Docker Compose)
- (Optional) IntelliJ IDEA / Rider / VS Code

---

## Clone the orchestrator (recommended)

### Option A: one command (best)
```bash
git clone --recurse-submodules https://github.com/CountryBoy0ner/Microservices-Orchestrator.git
cd Microservices-Orchestrator
```

### Option B: clone first, then init submodules
```bash
git clone https://github.com/CountryBoy0ner/Microservices-Orchestrator.git
cd Microservices-Orchestrator
git submodule update --init --recursive
```

---

## Open in IntelliJ IDEA

Open the **root folder**:
- `File → Open → Microservices-Orchestrator`

IDEA will detect that submodule folders are separate Git repositories. That is expected.

---

## Pull updates (day-to-day)

### 1) Pull orchestrator changes
```bash
git pull
```

### 2) Ensure submodules are checked out (exact SHAs pinned by orchestrator)
```bash
git submodule update --init --recursive
```

This will put each service at the exact commit tracked by the orchestrator.

---

## Update submodules to their latest remote commits (optional)

Sometimes you want the orchestrator to track the newest commits from each service branch (e.g., `develop`).

From the orchestrator root:
```bash
git submodule update --remote --merge
```

Then commit the new pointers:
```bash
git add Authentication-Service User-Service Order-Service Payment-Service API-Gateway
git commit -m "Bump submodules"
git push
```

> Note: This updates the orchestrator’s pointers. It does **not** automatically publish changes to service repos.

---

## Make changes to a service (standard workflow)

### Step 1: enter the service folder
```bash
cd User-Service
```

### Step 2: checkout the right branch and pull
```bash
git checkout develop
git pull
```

### Step 3: edit code and commit inside the service repo
```bash
git add .
git commit -m "Describe your change"
git push
```

### Step 4: update the orchestrator pointer
Go back to the orchestrator root and commit the updated submodule reference:
```bash
cd ..
git add User-Service
git commit -m "Update User-Service submodule"
git push
```

---

## Run everything with Docker Compose

From orchestrator root:
```bash
docker compose up -d --build
```

To view logs:
```bash
docker compose logs -f
```

To stop:
```bash
docker compose down
```

To rebuild one service:
```bash
docker compose build <service-name>
docker compose up -d <service-name>
```

---

## Add a new microservice as a submodule

From orchestrator root:
```bash
git submodule add <REPO_URL> <FOLDER_NAME>
git submodule update --init --recursive
git commit -m "Add <FOLDER_NAME> submodule"
git push
```

Example:
```bash
git submodule add https://github.com/your-org/New-Service.git New-Service
```

---

## Remove a submodule cleanly

From orchestrator root:
```bash
git submodule deinit -f -- <FOLDER_NAME>
git rm -f --cached <FOLDER_NAME>
rm -rf .git/modules/<FOLDER_NAME> <FOLDER_NAME>
git commit -m "Remove <FOLDER_NAME> submodule"
git push
```

(Windows PowerShell replaces `rm -rf` with `Remove-Item -Recurse -Force`.)

---

## Troubleshooting

### “No url found for submodule path … in .gitmodules”
This usually means a repository contains a **broken nested gitlink** (a leftover submodule entry without proper `.gitmodules` mapping).

Fix:
- Remove nested gitlinks inside that repository (example shown for a repo named `API-Gateway`):
```bash
cd API-Gateway
git ls-files --stage | grep 160000
# remove the listed paths, e.g.:
git rm -r --cached SomeNestedPath
git commit -m "Remove broken nested gitlinks"
git push
```

Then return to orchestrator and update the pointer:
```bash
cd ..
git add API-Gateway
git commit -m "Update API-Gateway submodule"
git push
```

### “fatal: not a git repository”
You are probably not in the orchestrator root folder.  
Run:
```bash
pwd
```
(or `cd` to the correct folder) and try again.

### CRLF/LF warning on Windows
Warnings like:
- `LF will be replaced by CRLF`

are common on Windows and usually harmless. If needed, you can configure line endings:
```bash
git config --global core.autocrlf true
```

---

## Best practices

- Keep infra files (compose, docs) in orchestrator.
- Keep code changes in each service repo.
- Always commit submodule pointer updates in orchestrator after pushing service changes.
- Prefer cloning with `--recurse-submodules`.

---

## Quick command cheat sheet

Clone + submodules:
```bash
git clone --recurse-submodules https://github.com/CountryBoy0ner/Microservices-Orchestrator.git
```

Pull orchestrator + submodules:
```bash
git pull
git submodule update --init --recursive
```

Update submodules to latest remote and commit pointers:
```bash
git submodule update --remote --merge
git add Authentication-Service User-Service Order-Service Payment-Service API-Gateway
git commit -m "Bump submodules"
git push
```

Run with Docker:
```bash
docker compose up -d --build
```
