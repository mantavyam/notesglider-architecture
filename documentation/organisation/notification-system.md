---
icon: bell-ring
---

# Notification System

## SECTION 22 — NOTIFICATION SYSTEM

### 22.1 Two-Tier Notification Architecture

All notifications in the system fall into one of two tiers:

| Tier              | Channel        | Default State                                                      | Description                                   |
| ----------------- | -------------- | ------------------------------------------------------------------ | --------------------------------------------- |
| **High Priority** | Email + In-App | Always ON (Email enabled by default)                               | Critical events requiring immediate attention |
| **Low Priority**  | Email + In-App | In-App ON by default; Email ON by default for High, opt-in for Low | All other pipeline and system events          |

### 22.2 Complete Event-to-Notification Mapping

| Event                                            | Recipient                  | Tier | Default Channels      |
| ------------------------------------------------ | -------------------------- | ---- | --------------------- |
| PDF output delivered to Teacher                  | Teacher                    | High | Email + In-App        |
| New Organisation submitted for approval          | Super Admin                | High | Email + In-App        |
| Organisation approved                            | Teacher                    | Low  | Email + In-App        |
| Organisation rejected                            | Teacher                    | High | Email + In-App        |
| Organisation assigned to Editor                  | Editor                     | Low  | Email + In-App        |
| Organisation re-assigned to new Editor           | New Editor, Old Editor     | High | Email + In-App        |
| New Editor account created                       | Editor                     | High | Email + In-App        |
| Editor account authorised                        | Editor                     | High | Email + In-App        |
| Editor account suspended                         | Editor                     | High | Email + In-App        |
| New Teacher account pending Editor authorisation | Editor                     | Low  | Email + In-App        |
| Teacher account authorised by Editor             | Teacher                    | High | Email + In-App        |
| Teacher account rejected by Editor               | Teacher                    | High | Email + In-App        |
| Sub-member pending Teacher authorisation         | Teacher                    | Low  | Email + In-App        |
| Sub-member account authorised                    | Sub-member                 | Low  | Email + In-App        |
| Document submitted to Editor queue               | Editor                     | Low  | Email + In-App        |
| Editor began reviewing document                  | Teacher (informational)    | Low  | In-App (Email opt-in) |
| Stage 1 SLA — 30 min remaining warning           | Editor                     | Low  | Email + In-App        |
| Stage 1 SLA elapsed — auto-continuation          | Editor                     | Low  | Email + In-App        |
| PDF generated — ready for Editor review          | Editor                     | Low  | Email + In-App        |
| Stage 2 SLA elapsed — auto-delivery              | Editor                     | Low  | Email + In-App        |
| Teacher flagged PDF for revision                 | Editor                     | High | Email + In-App        |
| Sub-member joined Organisation                   | Teacher's Editor           | Low  | Email + In-App        |
| Sub-member left Organisation                     | Teacher's Editor           | Low  | Email + In-App        |
| Invoice auto-generated for review                | Super Admin                | Low  | Email + In-App        |
| Invoice approved and sent                        | Teacher                    | Low  | Email + In-App        |
| Drive sync failure                               | Editor, Super Admin        | Low  | Email + In-App        |
| Storage limit warning (Drive or Neon)            | Affected user, Super Admin | High | Email + In-App        |
| Cloudinary upload failure (retry 3)              | Teacher                    | Low  | Email + In-App        |
| WeasyPrint failure escalation                    | Editor, Super Admin        | High | Email + In-App        |
| Image placeholder auto-generated                 | Teacher (post-generation)  | Low  | Email + In-App        |
| Payment cleared — access restored                | Teacher                    | Low  | Email + In-App        |
| Quarterly/Bi-Annual/Annual trigger fired         | Editor                     | Low  | Email + In-App        |
| Audit log deletion completed                     | Super Admin                | Low  | Email + In-App        |
| Billing cycle closed                             | Super Admin                | Low  | Email + In-App        |
| PREPAID invoice issued                           | Teacher                    | High | Email + In-App        |

### 22.3 User Notification Controls

* Both Email and In-App channels are individually toggleable per notification event type.
* Located in: Account Settings -> Notifications.
* The table above shows defaults — users can override any row except the high-priority Teacher PDF delivery (which cannot be fully disabled for the Teacher).
* **Email for Low Priority events**: Users can enable Email notifications for Low Priority events from their notification settings. By default, Low Priority events are In-App only, but the user can opt-in to receive them via Email as well.
* Sub-members can manage their own notification preferences independently of the Teacher's configuration for them.
