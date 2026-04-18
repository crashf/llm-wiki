---
title: Jellyfin Setup
type: summary
tags: [media-server, jellyfin, self-hosted, docker]
sources: 1
created: 2026-04-07
updated: 2026-04-07
---

## Summary

Jellyfin is a self-hosted media server running as a Docker container on a dedicated host. It provides media organization, streaming, and library management for Movies and TV Shows libraries. Access is via reverse proxy (https://jf.tech-method.com) with API key authentication.

## Connection Details

| Property | Value |
|----------|-------|
| Host | 170.205.18.108 (SSH port 1022) |
| Docker Container | `jellyfin` |
| Web UI | https://jf.tech-method.com (via NPM) |
| Local URL | http://localhost:8096 |
| API Key | `4e10f71f61ce4ecd939da544bf2e7b8f` |
| Admin User | crash (user ID: 56a3403bbc1f497ca728358999432e8d) |
| Config Path | /home/wayne/jellyfin/config |
| Database | /home/wayne/jellyfin/config/data/jellyfin.db (SQLite, ~1.4GB) |

## Libraries

| Library | ID | Content Type |
|---------|-----|--------------|
| Movies | f137a2dd21bbc1b99aa5c0f6bf02a805 | Movies |
| Shows | a656b907eb3a73532e40e44b968d0225 | TV Shows |

## Architecture

- **Deployment**: Docker container on bare metal (Ubuntu)
- **Reverse Proxy**: Nginx Proxy Manager (170.205.18.11:81) terminates SSL
- **Storage**: Media files on NFS mount, config on host
- **Database**: SQLite database at config path

## Configuration Notes

- API key authentication for programmatic access
- User ID needed for admin operations
- Library IDs required for library-specific API calls
- Config directory mounted from host for persistence

## Connections

- Related to [[jellyfin-api-patterns]] — API usage patterns and limitations
- Related to [[timezone-conventions]] — UTC/EDT handling learned from trickplay migration