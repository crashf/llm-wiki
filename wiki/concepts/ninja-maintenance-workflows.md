---
title: Ninja Maintenance Workflows
type: concept
tags: [ninja-rmm, maintenance-window, automation, patching, windows-server]
sources: 1
created: 2026-04-07
updated: 2026-04-07
---

## Summary

The 4-loop pattern is the core architectural pattern for NinjaRMM maintenance window automation. Each loop implements wait-and-check logic that polls device activity logs to detect phase completion before proceeding to the next phase.

## Key Points

- **4 loops:** Reboot → Scan → Install → Final Reboot
- **Polling:** Device activity log (`PATCH_MANAGEMENT` activity type)
- **Wait interval:** 5 minutes per check cycle
- **Completion detection:** Activity status codes (`PATCH_MANAGEMENT_SCAN_COMPLETED`, `PATCH_MANAGEMENT_APPLY_PATCH_COMPLETED`)
- **Failure handling:** Early exit via Switch node routing

## Connections

- Related to [[n8n-maintenance-automation]] — the 4-loop pattern implementation
- Related to [[n8n-design-patterns]] — n8n-specific implementation details
- See also [[llm-wiki-pattern]] — pattern for documenting automation projects

## Details

### The 4-Loop Pattern

Each loop follows the same structure:

```
Trigger Action → Wait 5min → GET Activities → Evaluate → Route (Success/Failure)
```

#### Loop 1: Reboot Phase
- **Action:** `POST /v2/device/{id}/reboot/NORMAL`
- **Check:** Device online status (not activity-based)
- **Success:** All devices respond to health check
- **Failure:** Report immediately

#### Loop 2: Scan Phase
- **Action:** `POST /v2/device/{id}/patch/os/scan`
- **Check:** Activities for `PATCH_MANAGEMENT_SCAN_COMPLETED` with `activityResult=SUCCESS`
- **Failure detection:** `PATCH_MANAGEMENT_FAILURE` with `activityResult=FAILURE`
- **Success:** Proceed to install phase
- **Failure:** Route to Build Report (early exit)

#### Loop 3: Install Phase
- **Action:** `POST /v2/device/{id}/patch/os/apply`
- **Check:** Activities for `PATCH_MANAGEMENT_APPLY_PATCH_COMPLETED` with `activityResult=SUCCESS`
- **Failure detection:** `PATCH_MANAGEMENT_INSTALL_FAILED` with `activityResult=FAILURE`
- **Success:** Proceed to final reboot
- **Failure:** Route to Build Report (early exit)

#### Loop 4: Final Phase
- **Action:** `POST /v2/device/{id}/reboot/NORMAL` (post-patch reboot)
- **Check:** Activities for `PATCH_MANAGEMENT_SCAN_COMPLETED` (final verification)
- **Success:** Trigger final scan → Build Report
- **Failure:** Route to Build Report (early exit)

### Activity-Based Detection

The key insight is using device activity logs instead of patch status endpoints:

```bash
GET /v2/device/{id}/activities?activityType=PATCH_MANAGEMENT&pageSize=50
```

The Code node evaluates:
- **scanComplete:** `statusCode=PATCH_MANAGEMENT_SCAN_COMPLETED AND activityResult=SUCCESS`
- **installComplete:** `statusCode=PATCH_MANAGEMENT_APPLY_PATCH_COMPLETED AND activityResult=SUCCESS`
- **hasFailure:** Any PATCH_MANAGEMENT_FAILURE or PATCH_MANAGEMENT_INSTALL_FAILED with activityResult=FAILURE
- **proceed:** `scanComplete && !hasFailure`
- **keepWaiting:** `!proceed && !hasFailure`

### Device Discovery

- Filter by `nodeClass === 'WINDOWS_SERVER'`
- Get all devices via `GET /v2/organization/{orgId}/devices`
- Remove hardcoded test device filters for production use

### API Requirements

- **Scope:** `management` on NinjaOne API client
- **Endpoints:** reboot, patch/scan, patch/apply, activities
- **Credential:** NinjaOne API (credential ID: `79Y08MOUWAdo6geq`)