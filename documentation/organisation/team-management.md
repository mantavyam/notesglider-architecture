---
icon: user-group
---

# Team Management

## SECTION 21 — TEAM & SUB-MEMBER MANAGEMENT

### 21.1 Team Overview

A Teacher can create an organisation and invite sub-members (colleagues, assistants) to join it. Sub-members can help create and manage documents within the Teacher's document pool.

### 21.2 Invite Flow

1. Teacher navigates to **Account Settings → Team Management**.
2. Teacher enters the email address of the person they want to invite.
3. Teacher configures initial permissions for this person (see Section 4.5).
4. Teacher clicks "Send Invite."
5. Invitee receives an email with an accept link.
6. Invitee clicks the link → creates an account (or logs in with existing account) → automatically joins the Teacher's organisation.
7. Sub-member's `status: active` immediately upon acceptance.
8. Teacher's assigned Editor receives a notification-only alert (no action needed).

### 21.3 Sub-member Permission Configuration

**Permission Panel Location**: Teacher Account Settings → Team Management → \[Sub-member Name] → Permissions

| Permission          | Toggle State | Effect                                            |
| ------------------- | ------------ | ------------------------------------------------- |
| Document Creation   | ON/OFF       | Can create new Newsletter documents               |
| Document Editing    | ON/OFF       | Can edit existing documents in the Teacher's pool |
| Document Submission | ON/OFF       | Can submit documents to the Editor's review queue |
| Export              | ON/OFF       | Can export documents (PPTX, PDF, TXT, ZIP)        |

* Permissions are enforced server-side on every API endpoint. Client-side hiding is supplementary.
* Teacher can update any sub-member's permissions at any time.
* Changes take effect immediately on the next request from that sub-member.

### 21.4 Sub-member Notification Settings

* Teacher controls whether each sub-member receives:
  * Email notifications for pipeline events
  * Final PDF output email delivery
* Sub-members can independently manage their own notification preferences in their Account Settings.
* Sub-member's own preference takes precedence for their individual account — it overrides the Teacher's setting for that person.

### 21.5 Leave Organisation Flow

1. Sub-member navigates to Account Settings → Request to Leave Organisation.
2. Teacher receives an in-app notification: _"\[Sub-member Name] has requested to leave your organisation."_ with Accept/Deny buttons.
3. On Teacher confirmation: Sub-member's access to Teacher's document pool is revoked immediately.
4. Sub-member's account remains active in the system (they can be re-invited later or join another organisation).
5. Teacher's assigned Editor receives a notification-only alert (no action needed).

### 21.6 Team Activity Log

* Located in Teacher Dashboard → Team Activity.
* Displays all sub-member actions with: action type, document affected, timestamp, sub-member name.
* Filterable by sub-member name, action type, date range.
* Read-only — Teacher cannot modify log entries.
