---
icon: ticket-simple
---

# Support Tickets

## SECTION 22A — CUSTOMER SUPPORT & TICKET SYSTEM

### 22A.1 Overview

Notesglider includes a built-in **customer support ticket system** accessible to all tenant-level roles (Teacher, Editor, Sub-member). This provides a structured mechanism for issue resolution, feature requests, and communication with the platform owner (Super Admin) — modelled after production support systems like Zendesk, Freshdesk, and Intercom, adapted for Notesglider's role hierarchy.

### 22A.2 Who Can Raise Tickets

| Role        | Can Raise Tickets          | Can View Tickets        | Can Resolve Tickets            |
| ----------- | -------------------------- | ----------------------- | ------------------------------ |
| Super Admin | No (receives and resolves) | All tickets system-wide | Yes — sole resolver            |
| Editor      | Yes                        | Own tickets only        | No — only Super Admin resolves |
| Teacher     | Yes                        | Own tickets only        | No — only Super Admin resolves |
| Sub-member  | Yes                        | Own tickets only        | No — only Super Admin resolves |

### 22A.3 Ticket Creation Flow

1. User navigates to their **RBAC Dashboard → Settings → Support → "Create New Ticket."**
2. Fills in the ticket form:

| Field            | Type                                     | Required | Description                                                                                                                       |
| ---------------- | ---------------------------------------- | -------- | --------------------------------------------------------------------------------------------------------------------------------- |
| Subject          | String (max 200 chars)                   | Yes      | Brief description of the issue                                                                                                    |
| Category         | Enum                                     | Yes      | `bug-report`, `feature-request`, `billing-inquiry`, `account-issue`, `technical-support`, `compliance-dispute`, `general-inquiry` |
| Priority         | Enum                                     | Yes      | `low`, `medium`, `high`, `critical`                                                                                               |
| Description      | Text (max 5000 chars)                    | Yes      | Detailed description of the issue                                                                                                 |
| Attachments      | File upload (max 5 files, max 10MB each) | No       | Screenshots, PDFs, or other supporting files                                                                                      |
| Related Document | Document ID (nullable)                   | No       | Link to a specific document if the issue is document-related                                                                      |

3. On submission:
   * Ticket is created with `status: open`.
   * **Super Admin receives**:
     * Email notification: _"New support ticket #{ticket\_number} from \[{Role}] \[{User Name}] ({Organisation Name}): \[{Subject}]"_
     * In-app notification in the Super Admin dashboard.
   * **User receives**: Confirmation email + in-app notification: _"Your support ticket #{ticket\_number} has been submitted. We will respond as soon as possible."_

### 22A.4 Ticket Lifecycle States

```
open → in-progress → awaiting-user-response → in-progress → resolved → closed
                                             ↘ escalated → in-progress → resolved → closed
open → resolved (direct resolution) → closed
```

| Status                   | Description                                                    | Set By                                                     |
| ------------------------ | -------------------------------------------------------------- | ---------------------------------------------------------- |
| `open`                   | Newly created, awaiting Super Admin attention                  | System (on creation)                                       |
| `in-progress`            | Super Admin is actively working on the ticket                  | Super Admin                                                |
| `awaiting-user-response` | Super Admin has replied and is waiting for the user's response | Super Admin                                                |
| `escalated`              | Ticket requires elevated attention or external investigation   | Super Admin                                                |
| `resolved`               | Issue has been addressed; pending user confirmation            | Super Admin                                                |
| `closed`                 | Ticket is permanently closed (auto or manual)                  | System (14-day auto-close after resolution) or Super Admin |
| `reopened`               | User reopens a resolved ticket with additional context         | User (ticket creator)                                      |

#### Auto-Close Policy

* Resolved tickets are automatically closed after **3 days** with no further user response.
* The user receives a reminder notification at day 2: _"Your support ticket #{ticket\_number} was marked as resolved. It will be automatically closed tommorow. If the issue persists, please reopen the ticket."_

### 22A.5 Ticket Communication Thread

Each ticket has a **conversation thread** — a chronological exchange between the ticket creator and the Super Admin.

* **Thread messages** support: plain text (max 2000 chars per message) and file attachments (max 3 files, max 10MB each per message).
* Each message displays: author name, role badge, timestamp, message body, and attachments (if any).
* The ticket creator and Super Admin can both add messages.
* **Visibility**: Only the ticket creator and the Super Admin can see the ticket's conversation thread. No other role has access to another user's tickets.
* **Email forwarding**: Each new message in the thread triggers an email notification to the other party, including the message body in the email for convenience.

### 22A.6 Super Admin Ticket Dashboard

The Super Admin Dashboard includes a **Support** tab with a full ticket management interface:

**Interface:**

* **Inbox view**: All tickets ordered by priority (critical first), then creation date (oldest first within each priority).
* **Tabs**: `Open`, `In Progress`, `Awaiting Response`, `Escalated`, `Resolved`, `Closed`.
* **Filters**: Category, priority, role of reporter, Organisation, date range.
* **Search**: Full-text search across ticket subjects and descriptions.
* **Bulk actions**: Mark multiple tickets as resolved, change priority, reassign category.

**Per-Ticket Actions:**

* Reply (add message to thread)
* Change status
* Change priority
* Add internal note (visible only to Super Admin, not to the ticket creator — for personal bookkeeping)
* Link to related tickets
* Export ticket thread as PDF

**Metrics Panel:**

* Total open tickets (by priority breakdown)
* Average resolution time (by category)
* Tickets created this week / month
* Most common categories (bar chart)

### 22A.7 Ticket from Settings UI

Each role accesses the ticket system from their respective Settings page:

* **Teacher**: Account Settings → Support → My Tickets (list) + "Create New Ticket"
* **Editor**: Account Settings → Support → My Tickets (list) + "Create New Ticket"
* **Sub-member**: Account Settings → Support → My Tickets (list) + "Create New Ticket"

Each user sees only their own tickets with the current status and the option to view the conversation thread.

### 22A.8 Data Retention

* Ticket data (including conversation threads and attachments) follows the **same retention and archival policy as audit logs** (see Section 8.0.3).
* The Super Admin can manage ticket data deletion through the same audit log deletion interface.
* Before deletion: ticket data is archived to Google Drive and exportable as CSV/PDF.
* Ticket attachments are stored in the application's file storage and follow the same archival lifecycle.
