# Release Notes

## Overview
This document records the actual diagnoses, fixes, commands, verification steps, remaining risks, and rebuild procedures for the CloudNotes container deployment project.

## Tasks Completed

### Task 1 - Image Build Repair
- **Diagnosis:** Dockerfile referenced non-existent file `requirements-missing.lock` on line 3
- **Fix Applied:** Changed `COPY requirements-missing.lock` to `COPY requirements.lock`
- **Commands Used:** `docker build -t container-deployment:1.0.0 .`
- **Verification:** Build succeeds without "no such file" error

### Task 2 - Local Image Delivery
- **Diagnosis:** Image tag in load-image.sh had typo `1.0.O` (letter O) instead of `1.0.0` (zero)
- **Fix Applied:** Changed line 3 from `IMAGE=container-deployment:1.0.O` to `IMAGE=container-deployment:1.0.0`
- **Commands Used:** `./scripts/load-image.sh` and `kind load docker-image container-deployment:1.0.0`
- **Verification:** Image successfully loaded to cluster without tag mismatch error

### Task 3 - Service Routing
- **Diagnosis:** Service selector `app: cloudnote` didn't match Pod label `app: cloudnotes` (missing 's')
- **Fix Applied:** Changed k8s/service.json line 10 selector from `"cloudnote"` to `"cloudnotes"`
- **Commands Used:** `kubectl -n container-deployment get endpointslice` and `kubectl describe service cloudnotes`
- **Verification:** Service now has matching endpoints and routes traffic to Pods

### Task 4 - Configuration Consumption
- **Diagnosis:** Deployment referenced ConfigMap key `AP_MODE` instead of actual key `APP_MODE` (missing 'P')
- **Fix Applied:** Changed k8s/deployment.json line 51 from `"key": "AP_MODE"` to `"key": "APP_MODE"`
- **Commands Used:** `kubectl describe pod` and `curl http://127.0.0.1:18082/info`
- **Verification:** Pod starts successfully and /info endpoint shows mode: "assessment"

### Task 5 - Release Verification Gate
- **Diagnosis:** Script had `|| true` on line 4, always returning exit code 0 regardless of test results
- **Fix Applied:** Removed `|| true` from scripts/verify-release.sh line 4
- **Commands Used:** `BASE_URL=http://127.0.0.1:1 ./scripts/verify-release.sh; echo $?` and `./scripts/verify-release.sh; echo $?`
- **Verification:** Script fails (exit != 0) on unreachable endpoint, succeeds (exit = 0) on healthy service

## Remaining Risks
- Local cluster must have sufficient resources (4 CPU, 8GB RAM minimum)
- PersistentVolumeClaim requires dynamic provisioner configured in cluster
- Single replica with Recreate strategy - no high availability
- SQLite database doesn't support concurrent writers (intentional for this exercise)
- Port-forward required for local testing (no Ingress configured)

```
