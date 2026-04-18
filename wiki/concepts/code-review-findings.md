# Code Review Findings - Club Sixty Six

**Date:** 2026-04-15

## Overview

Comprehensive code review of Club Sixty Six backend codebase by subagent. Focused on server actions, route handlers, mutations, and CRUD operations.

---

## Priority Matrix

| Priority | Issue | File | Impact |
|----------|-------|------|--------|
| 🔴 HIGH | Blog slug race condition | `admin/blog/route.ts` | Concurrent POSTs can duplicate slugs |
| 🔴 HIGH | Duplicate requireAdmin() | Multiple files | Missing user existence checks |
| 🔴 HIGH | Unvalidated form data | `admin/upload/route.ts` | Type validation bypasses |
| 🔴 HIGH | DB before files | `admin/videos/bulk-delete` | Orphaned files on failure |
| 🟠 MEDIUM | HLS files not deleted | `admin/videos/[id]` | Storage leak |
| 🟠 MEDIUM | Inconsistent slug logic | Multiple upload routes | Code duplication |
| 🟠 MEDIUM | No transaction wrapping | `admin/programs/[id]/videos` | Race conditions |
| 🟠 MEDIUM | Silent bulk failures | Multiple bulk routes | Poor error reporting |

---

## Technical Details

### High Priority Details

#### 1. TOCTOU Race Condition
```typescript
// admin/blog/route.ts:42-47
const baseSlug = slugify(title);
let slug = baseSlug;
let counter = 1;
while (await prisma.blogPost.findUnique({ where: { slug } })) {
  slug = `${baseSlug}-${counter++}`;
}
```

**Issue:** Two simultaneous requests with same title both pass uniqueness check before insert.

**Fix:** Use database-level unique constraint handling with transaction retry.

#### 2. Inline Auth Implementations

Multiple files inline this pattern:
```typescript
async function requireAdmin(userId: string) {
  const user = await prisma.user.findUnique({ where: { id: userId } });
  if (!user || user.role !== "ADMIN") {
    throw new Error("Unauthorized");
  }
}
```

**Issue:** Some implementations don't check `if (!user)` properly or handle null cases.

**Fix:** Import from centralized `@/lib/admin-auth` to ensure consistency.

#### 3. Form Data Validation

```typescript
// admin/upload/route.ts
const durationStr = (formData.get("duration") as string) || "";
```

**Issue:** No type validation - assumes string, could be File or null.

**Fix:** Validate with `typeof` or `zod` schema validation.

#### 4. DB-First Deletion Pattern

```typescript
// admin/videos/bulk-delete/route.ts:30-43
await prisma.video.deleteMany({ where: { id: { in: videoIds } } });
// Then try to cleanup files - if this fails, orphaned files remain
```

**Issue:** DB modified before file cleanup, leaving orphaned files on disk.

**Fix:** Transaction pattern - delete files first, DB second, rollback on file failure.

---

## References

- Full Todo List: `projects/ClubSixtySix/CODE-REVIEW-TODO-2026-04-15.md`
- Related: [Club Sixty Six Infrastructure](project-clubsixtysix.md)
