---
title: Jellyfin API Patterns
type: concept
tags: [jellyfin, api, sqlite, troubleshooting, media-server]
sources: 1
created: 2026-04-07
updated: 2026-04-07
---

## Summary

Jellyfin provides a REST API for most operations, but certain tasks (like episode updates) require direct SQLite database manipulation. The API has limitations, particularly with Episode-type items where POST requests to /Items/{id} return 500 errors.

## REST API Usage

### Authentication
- Use API key in header: `X-MediaBrowser-Token: 4e10f71f61ce4ecd939da544bf2e7b8f`
- Admin user ID for user-specific operations: `56a3403bbc1f497ca728358999432e8d`

### Common Endpoints
- `/Items` — library items
- `/Shows` — TV show metadata
- `/Items/{id}` — individual item operations
- `/Library/Selectable` — available libraries

### Known Limitations

| Operation | REST API | Direct DB |
|-----------|----------|-----------|
| Update Movie metadata | ✅ Works | ✅ Works |
| Update TV Show metadata | ✅ Works | ✅ Works |
| Update Episode metadata | ❌ Returns 500 | ✅ Works |

**Critical Issue**: POST requests to `/Items/{episodeId}` for Episode types return HTTP 500 errors. The API cannot update episode-level metadata through standard REST calls.

## Direct Database Access

When API fails, modify SQLite database directly:

### Database Location
```
/home/wayne/jellyfin/config/data/jellyfin.db (~1.4GB)
```

### Common Direct DB Operations
- Episode metadata updates (season, episode number, air date)
- Trickplay thumbnail generation status
- Playback position fixes
- Metadata corrections that fail via API

### Best Practices
1. **Backup first**: `cp jellyfin.db jellyfin.db.backup`
2. **Use SQLite CLI**: `sqlite3 jellyfin.db`
3. **Verify changes**: Query before/after to confirm update worked
4. **Restart container**: `docker restart jellyfin` to clear caches

## Timezone Handling (Critical Lesson)

**Never confuse UTC with local time in Jellyfin operations.**

### The Trickplay Migration Incident (2026-03-21)

During a trickplay (preview thumbnails) migration:
- Migration script was scheduled to run at "midnight"
- It ran at **8 PM EDT** instead of midnight
- **Root cause**: Script used UTC time but operator assumed EDT

### Timezone Math

| Local Time (EDT/EST) | UTC Offset | UTC Time |
|---------------------|-------------|----------|
| Midnight EDT (summer) | UTC-4 | 04:00 UTC |
| Midnight EST (winter) | UTC-5 | 05:00 UTC |
| 8 PM EDT | UTC-4 | 00:00 UTC (next day) |

**Lesson**: Midnight EDT ≠ Midnight UTC. In EDT (summer), midnight local = 04:00 UTC. A task scheduled for "midnight UTC" fires at 8 PM EDT the previous day.

### Best Practice
- **Always specify timezone** in schedules: "midnight America/New_York" or "00:00 UTC"
- **Confirm with timezone converter** before scheduling critical jobs
- **Document expected vs actual** when issues arise

## Connections

- Related to [[jellyfin-setup]] — server configuration details
- Related to [[timezone-conventions]] — broader timezone handling rules