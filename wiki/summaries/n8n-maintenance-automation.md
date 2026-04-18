---
title: N8N Maintenance Automation
type: summary
tags: [n8n, ninja-rmm, automation, patching, windows-server]
sources: 1
created: 2026-04-07
updated: 2026-04-07
---

## Summary

N8N workflow that automates Windows Server patching via NinjaRMM API. It performs device discovery, orchestrated reboots, patch scanning, patch application, post-patch verification, and final reporting across all Windows Servers in an organization. Uses activity-based polling to detect completion status rather than relying on patch status endpoints.

## Key Points

- **Workflow ID:** `b4ZdOXrgjoQWQ330` on n8n.pund-it.ca
- **Discovery:** Filters devices by `nodeClass === 'WINDOWS_SERVER'`
- **Phases:** Reboot → Scan → Install → Final Reboot → Final Scan
- **Detection Method:** Device activity log polling (`GET /v2/device/{id}/activities?activityType=PATCH_MANAGEMENT&pageSize=50`)
- **Loop Logic:** 5-minute wait intervals with activity evaluation for completion detection
- **Failure Handling:** Switch nodes route failures to early exit report generation

## Connections

- Related to [[n8n-design-patterns]] — documents the critical typeVersion bug discovered during development
- Related to [[ninja-maintenance-workflows]] — defines the 4-loop pattern used in this automation
- See also [[llm-wiki-pattern]] — this wiki structure was used to document the project

## Details

### Workflow Architecture (30 nodes)

The workflow progresses through distinct phases, each with its own wait-and-check loop:

1. **Discovery Phase:** Get all org devices, filter to Windows Servers
2. **Reboot Phase:** Reboot all devices, wait 5min, check online status
3. **Scan Phase:** Trigger OS patch scan, wait 5min, check activities for `PATCH_MANAGEMENT_SCAN_COMPLETED`
4. **Install Phase:** Apply patches, wait 5min, check activities for `PATCH_MANAGEMENT_APPLY_PATCH_COMPLETED`
5. **Final Phase:** Post-patch reboot, verify, trigger final scan, build report

### API Endpoints Used

- `POST /v2/device/{id}/reboot/NORMAL` — Reboot device
- `POST /v2/device/{id}/patch/os/scan` — Trigger patch scan
- `POST /v2/device/{id}/patch/os/apply` — Apply pending patches
- `GET /v2/device/{id}/activities?activityType=PATCH_MANAGEMENT&pageSize=50` — Poll completion status
- `GET /v2/organization/{orgId}/devices` — List org devices

### Key Lessons Learned

- Activity-based polling more reliable than patch status endpoints
- 5-minute wait intervals prevent premature progression
- Switch nodes enable early failure detection and reporting
- typeVersion 1 required for If nodes in API-based workflow creation
- Hardcoded test device filters must be removed for production use