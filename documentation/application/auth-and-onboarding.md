---
icon: user-shield
---

# Auth & Onboarding

## SECTION 5 — AUTHENTICATION & ONBOARDING

> **Implementation Note**: Authentication is implemented using **Better-Auth** (`better-auth` npm package) with the Prisma database adapter (`@better-auth/prisma-adapter`). Better-Auth provides Google OAuth, Magic Link, and Email+Password authentication with secure HttpOnly cookie session management. See [better-auth.com](https://www.better-auth.com/) for documentation.

### 5.1 Login Methods

All four user types (Super Admin, Editor, Teacher, Sub-member) use the same authentication system. The login page is shared; the system determines role and redirects to the correct dashboard based on the authenticated identity.

| Method               | Priority             | Notes                                                                                  |
| -------------------- | -------------------- | -------------------------------------------------------------------------------------- |
| **Google OAuth**     | Primary              | Required for Google Drive/Docs/Translate integration. Always the first visible option. |
| **Email + Password** | Fallback             | Email must be OTP-verified before first login is permitted.                            |
| **Magic Link**       | Alternative fallback | One-time login link sent to the user's verified email address.                         |

> **Super Admin Note**: The Super Admin account is **provisioned directly at system setup time** by the developer/platform owner — it is not created through any in-app self-registration flow. Super Admin credentials are seeded into the database during deployment. There is no public-facing registration path for the Super Admin role under any circumstances.

#### Google OAuth Permission Scopes

When a user authenticates via Google OAuth, the application requests **only** the following approved non-sensitive scopes:

* `openid` — Associate the user with their personal info on Google
* `.../auth/userinfo.email` — See the user's primary Google Account email address
* `.../auth/userinfo.profile` — See personal info, including any info made publicly available
* `.../auth/drive.file` — See, edit, create and delete only the specific Google Drive files used with this app
* `.../auth/drive.appdata` — See, create and delete its own configuration data in the user's Google Drive
* `.../auth/drive.install` — Connect itself to the user's Google Drive

> **YouTube scope removed**: The YouTube Data API scope is **not requested** for any role. YouTube video discovery uses a custom URL-based interface that requires no OAuth grant (see Section 27). This set of scopes has been verified by Google — no sensitive or restricted scopes are requested and verification is not required.

> **Scope Applicability**: Drive scopes apply to Teachers and Editors. Super Admins and Sub-members do not require Drive scopes. Scope requests must be role-aware — request only the scopes applicable to the authenticated role, not a blanket set for all.

If the user declines any applicable scope:

* The app must flag the specific missing permissions.
* Display a clear message explaining which functionality will not work without each declined scope.
* Include a reassurance message: _"Your data is encrypted and safe. We only access what you explicitly grant."_
* Allow the user to proceed with limited functionality (do not hard-block login for partial scope grants).
* Provide a settings screen to re-grant permissions later.

### 5.2 Session Persistence

* Once authenticated, users remain signed in persistently using secure token storage (HttpOnly cookies — never localStorage).
* Token refresh must happen transparently in the background — users must never be unexpectedly logged out during an active session.
* On token expiry failure, redirect to login with a clear notification (not a silent failure).
* Session scope is role-aware: session context carries `{ user_id, role, org_id (nullable for Super Admin), editor_id (nullable for Super Admin) }`.

### 5.3 Super Admin Account Provisioning (Developer-Only)

The Super Admin is **not self-registered** through the application. It is seeded at deployment time:

1. Developer runs a provisioning script (or migration) to create the Super Admin record in the `super_admins` table with a hashed password and `status: active`.
2. The Super Admin logs in via Email + Password or Google OAuth — no approval gate applies.
3. The Super Admin immediately has full system-level access upon first successful login.
4. There is no in-app UI for creating additional Super Admin accounts. If more than one platform owner needs Super Admin access, it must be done through direct database provisioning or a protected CLI command — never through the regular app UI.

### 5.4 Editor Onboarding Flow

Editors cannot self-register. Their accounts are created exclusively by the Super Admin.

```
Super Admin
  → Creates Editor account in Super Admin Dashboard
    (enters: name, email, role = editor)
    (system sends Editor a "Your account has been created" email with a password-set link)
  → Editor account created with status: pending
  → Super Admin explicitly Authorises the Editor account → status: active
  → Super Admin maps one or more Organisations to this Editor
  → Editor can now log in and begin operating
```

**Step-by-Step:**

**Step 1 — Super Admin Creates Editor**

* Super Admin navigates to Super Admin Dashboard → Editor Management → "Create New Editor."
* Enters: Editor's name, email address.
* System creates the Editor record with `status: pending`.
* System sends the Editor an email: _"Your Notesglider Editor account has been created. Click here to set your password and activate your account."_

**Step 2 — Super Admin Authorises Editor**

* Super Admin navigates to Editor Management → Pending Editors.
* Reviews the Editor record and clicks "Authorise."
* Editor `status` changes to `active`.
* Editor receives in-app and email notification: _"Your Editor account has been authorised. You can now log in to your dashboard."_

**Step 3 — Organisation Mapping (see Section 5.5, Stage 3)**

* Once an Organisation is approved, Super Admin maps it to this Editor (or any other active Editor).
* Only after mapping does the Editor gain visibility into that Organisation's Teachers and data.

### 5.5 Teacher & Organisation Onboarding Flow

Teachers **self-register** through the public-facing signup page. They simultaneously create their Organisation (tenant container) during registration. This is the key difference from the old invite-link model: Teachers initiate their own registration; the Super Admin and Editor gate their access.

```
Teacher self-registers + creates Org  →  org.status = pending_approval
Super Admin reviews + authorises Org  →  org.status = active
Super Admin maps Org to an Editor     →  org.editor_id = [assigned Editor]
Editor authorises Teacher account     →  teacher.status = active
Teacher gains full application access
```

**Step-by-Step:**

**Stage 1 — Teacher Self-Registration + Organisation Creation**

* Teacher navigates to the public signup page (no invite link required).
* Completes registration via any of the three authentication methods.
* During registration, Teacher fills in their **Organisation details**: Organisation name, description, region/timezone.
* On completion: Teacher account is created with `status: pending`. Organisation is created with `status: pending_approval`.
* Teacher sees a holding screen: _"Your registration has been submitted. Your account will be activated once your organisation is reviewed and approved."_
* Teacher has **zero application access** in this state.
* Super Admin receives an in-app + email notification: _"A new Organisation '\[Org Name]' has been submitted for approval by Teacher \[Name]."_

**Stage 2 — Super Admin Reviews & Authorises the Organisation**

* Super Admin navigates to Super Admin Dashboard → Organisation Management → Pending Approvals.
* Views Organisation details (name, Teacher identity, registration date, region).
* Clicks "Approve" or "Reject."
* **On Approval**: Organisation `status` → `active`. Moves to Stage 3.
* **On Rejection**: Teacher receives email notification with reason (if provided). Organisation and Teacher account are soft-deleted after a 30-day grace period.

**Stage 3 — Super Admin Maps Organisation to an Editor**

* Immediately following approval (or at any time after), the Super Admin maps the Organisation to an Editor.
* Super Admin navigates to Organisation Management → \[Org Name] → "Assign Editor."
* Selects from the list of active Editors.
* Organisation record updated: `editor_id = [selected Editor's ID]`.
* The assigned Editor receives an in-app + email notification: _"Organisation '\[Org Name]' (Teacher: \[Name]) has been assigned to you. Please review and authorise the Teacher's account."_

**Stage 4 — Editor Authorises the Teacher**

* Editor navigates to their RBAC Dashboard → Teacher Management → Pending Authorisations.
* Reviews Teacher details (name, email, Organisation, registration date).
* Clicks "Authorise" or "Reject."
* **On Authorisation**: Teacher `status` → `active`. Teacher receives in-app + email notification: _"Your account has been activated. You can now access your dashboard."_ Teacher gains full application access.
* **On Rejection**: Teacher receives email notification with reason (if provided). Account is soft-deleted.

> **Organisation Re-Assignment**: The Super Admin can reassign an Organisation from one Editor to another at any time — for example, if an Editor leaves the platform. On re-assignment, the new Editor gains full visibility and management access to the Organisation. The old Editor immediately loses access. All documents, billing data, and settings are preserved and transferred.

### 5.6 Sub-member Onboarding Flow

Sub-members are enrolled into a Teacher's Organisation by the Teacher. They can either self-register or receive an invite link.

```
Sub-member self-registers or receives invite  →  sub_member.status = pending
Teacher reviews and authorises sub-member     →  sub_member.status = active
Sub-member gains scoped application access
```

**Step-by-Step:**

**Stage 1 — Registration / Invite**

* _Path A (Invite)_: Teacher sends an invite link from Team Settings → Team Management → "Invite Member." Invitee receives an email with the link, clicks it, and completes account creation via any auth method.
* _Path B (Self-register)_: Sub-member registers independently and specifies the Organisation code/name they wish to join (or follows a public join link that encodes the `org_id`).
* In both paths: Sub-member account is created with `status: pending`. Sub-member cannot access any application functionality.

**Stage 2 — Teacher Authorises**

* Teacher receives an in-app notification: _"\[Sub-member Name] has requested to join your organisation. Please review and authorise their account."_
* Teacher navigates to Team Settings → Pending Members.
* Reviews and clicks "Authorise" or "Reject."
* **On Authorisation**: Sub-member `status` → `active`. Teacher assigns initial per-member permissions (see Section 21). Sub-member receives notification: _"Your account has been activated."_
* **On Rejection**: Sub-member receives email notification. Account is soft-deleted.

**Stage 3 — Scope Inheritance Notifications**

* The Teacher's assigned Editor receives a **notification-only** alert: _"\[Teacher Name]'s Organisation has a new sub-member: \[Sub-member Name]."_ No action required.
* The Super Admin's system-level activity log records the new sub-member creation event automatically.

### 5.7 Sub-member Leave Flow

* Sub-member navigates to Account Settings → Request to Leave Organisation.
* Teacher receives an in-app notification requiring confirmation.
* Teacher confirms or denies the leave request.
* On confirmation: Sub-member account is disassociated from the Organisation. All their document contributions remain — document records are not deleted. Access to Teacher's document pool is revoked immediately.
* The Teacher's assigned Editor receives a **notification-only** alert: _"\[Sub-member Name] has left \[Teacher Name]'s Organisation."_ No action required.
* The event is recorded in the Super Admin's system-level audit log.
