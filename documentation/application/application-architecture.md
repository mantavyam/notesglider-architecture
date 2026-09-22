---
icon: sitemap
---

# Application Architecture

## SECTION 7 — APPLICATION ARCHITECTURE OVERVIEW

### 7.1 Three-Tier Platform Mental Model

```
┌─────────────────────────────────────────────────────────────────────────┐
│                          NOTESGLIDER PLATFORM                           │
│                                                                         │
│  ┌──────────────────────────────────────────────────────────────────┐   │
│  │                  SUPER ADMIN LAYER  (System-Wide)                │   │
│  │  Editor Management · Organisation Approvals · Org→Editor Mapping │   │
│  │  System Logs · Platform Config · Account Suspension              │   │
│  └──────────────────────────────────────────────────────────────────┘   │
│                                ▼ governs                                │
│  ┌──────────────────────────────────────────────────────────────────┐   │
│  │                  EDITOR LAYER  (Tenant-Scoped)                   │   │
│  │  Review Queue · PDF Pipeline · Mindmap · Billing Dashboard       │   │
│  │  Drive Management · Teacher Authorisation · HTML Export          │   │
│  └──────────────────────────────────────────────────────────────────┘   │
│              ▼ send documents              ▼ deliver PDFs               │
│  ┌──────────────────────────────────────────────────────────────────┐   │
│  │                  TEACHER / ORG LAYER  (Org-Scoped)               │   │
│  │  Canvas Editor · PPTX Export · Presentation Mode · Translation   │   │
│  │  Team Management · Ad Banners · Templates                        │   │
│  └──────────────────────────────────────────────────────────────────┘   │
│                                ▼ sub-members                            │
│  ┌──────────────────────────────────────────────────────────────────┐   │
│  │             SUB-MEMBER LAYER  (Document Pool-Scoped)             │   │
│  │     Core document creation/editing per Teacher-granted perms     │   │
│  └──────────────────────────────────────────────────────────────────┘   │
│                                                                         │
│                    ┌────────────────────────┐                           │
│                    │  SHARED DATA LAYER     │                           │
│                    │  Neon DB (Prisma)      │                           │
│                    │  Cloudinary CDN        │                           │
│                    │  Google Drive          │                           │
│                    │  Real-time Sync        │                           │
│                    └────────────────────────┘                           │
└─────────────────────────────────────────────────────────────────────────┘
```

### 7.2 Core Architectural Rules

1. **Super Admin operates outside all tenant scopes.** The Super Admin's queries are unrestricted — they carry no `org_id` or `editor_id` predicate. No tenant can see or influence Super Admin operations.
2. **The Editor's scope is bounded by their mapped Organisations.** Every Editor API call is predicated on `WHERE org_id IN (SELECT id FROM organisations WHERE editor_id = current_editor_id)`. Cross-tenant access is architecturally impossible, not just UI-hidden.
3. **The Teacher's scope is bounded to their single Organisation.** Every Teacher API call is predicated on `WHERE org_id = current_teacher.org_id`. Teachers cannot reach data in other Organisations even if they know the IDs.
4. **PDF pipeline is invisible to Teachers.** No PDF-related UI element, button, or status should ever appear on any Teacher-facing screen. Enforced via RBAC middleware.
5. **Mindmap generation is invisible to Teachers.** Same rule — Editor-only.
6. **All backend API endpoints must validate the caller's role AND scope** before processing. Role checks on the frontend (showing/hiding UI) are UX conveniences — they do not replace server-side enforcement. The middleware chain must be: authenticate → resolve role → resolve scope → check permission → execute.
7. **Status-gated access is enforced at the middleware layer.** A `pending` account — at any role level — is rejected before any permission resolution. The `status` field is checked first, before any RBAC logic runs.
8. **Real-time sync** (Yjs CRDT — see §13.4 and §24) handles document co-visibility between Teacher and Editor during active review sessions.
9. **Audit log is append-only during normal operation.** Every authorization decision, account state change, and data modification event is written to the audit log. No record is ever edited or hard-deleted from the audit log except by the Super Admin through the managed audit log deletion process (see Section 8.0.3). Before any deletion, logs are archived to Google Drive and exportable as CSV.
