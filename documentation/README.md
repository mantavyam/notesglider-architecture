---
icon: hand-wave
metaLinks: {}
---

# Welcome

## Jump right in

<table data-view="cards"><thead><tr><th></th><th data-type="content-ref"></th></tr></thead><tbody><tr><td><i class="fa-star-shooting">:star-shooting:</i></td><td><a href="https://app.gitbook.com/s/Joa7EtMsaty0WdGC5FWR/getting-started">GETTING STARTED</a></td></tr><tr><td><i class="fa-computer">:computer:</i></td><td><a href="https://app.gitbook.com/s/Joa7EtMsaty0WdGC5FWR/application">APPLICATION</a></td></tr><tr><td><i class="fa-file">:file:</i></td><td><a href="https://app.gitbook.com/s/Joa7EtMsaty0WdGC5FWR/documents">DOCUMENTS</a></td></tr><tr><td><i class="fa-house">:house:</i></td><td><a href="https://app.gitbook.com/s/Joa7EtMsaty0WdGC5FWR/organisation">ORGANISATION</a></td></tr><tr><td><i class="fa-almost-equal-to">:almost-equal-to:</i></td><td><a href="https://app.gitbook.com/s/Joa7EtMsaty0WdGC5FWR/api">API</a></td></tr><tr><td><i class="fa-store">:store:</i></td><td><a href="https://app.gitbook.com/s/Joa7EtMsaty0WdGC5FWR/database">DATABASE</a></td></tr><tr><td><i class="fa-rocket-launch">:rocket-launch:</i></td><td><a href="https://app.gitbook.com/s/Joa7EtMsaty0WdGC5FWR/production">PRODUCTION</a></td></tr><tr><td><i class="fa-asterisk">:asterisk:</i></td><td><a href="https://app.gitbook.com/s/Joa7EtMsaty0WdGC5FWR/misc">MISC</a></td></tr></tbody></table>

## PRODUCT REQUIREMENTS DOCUMENT

### Final Developer Handoff Edition

***

| Field             | Value                              |
| ----------------- | ---------------------------------- |
| **Product Name**  | Notesglider                        |
| **Version**       | 5.0.0 — Final                      |
| **Document Type** | Full Product Requirements Document |
| **Industry**      | EduTech                            |
| **Core Function** | Document Automation SaaS           |
| **Last Updated**  | May 2026                           |
| **Status**        | Developer Handoff Ready            |

> **Binding Notice**: This document represents the complete, authoritative product specification. Every section — including all business rules, edge cases, error states, role boundaries, and integration details — has been explicitly confirmed by the product owner. The developer must implement exactly what is specified here. Nothing should be assumed, inferred, or left open for interpretation. Where a decision has two valid approaches, the specification below is the chosen one and must not be substituted.

### TABLE OF CONTENTS

* PRODUCT REQUIREMENTS DOCUMENT
  * Final Developer Handoff Edition
* [SECTION 1 — EXECUTIVE SUMMARY](getting-started/quickstart.md#section-1-executive-summary)
* [SECTION 2 — PRODUCT VISION & PROBLEM STATEMENT](getting-started/quickstart.md#section-2-product-vision-and-problem-statement)
  * 2.1 Educator Pain Points (Solved by Notesglider)
  * 2.2 Student Pain Points (Solved Indirectly)
  * 2.3 Core Product Principles
* [SECTION 3 — TECHNOLOGY STACK](getting-started/tech-stack.md)
  * 3.1 Complete Stack
  * 3.2 Key Library References
* [SECTION 4 — USER ROLES & RBAC SYSTEM](getting-started/rbac.md)
  * 4.1 Architecture: Multi-Tenant Hierarchically Delegated RBAC
    * The 5-Layer Role Hierarchy
  * 4.2 Login Role Types
  * 4.3 The Organisation — Tenant Container (Not a Role)
  * 4.4 Role 1: Super Admin — Capabilities
  * 4.5 Role 2: Editor — Capabilities
  * 4.6 Role 3: Teacher — Capabilities
  * 4.7 Role 4: Sub-member — Capabilities
  * 4.8 Sub-member Activity Logging
  * 4.9 Complete Permission Matrix
  * 4.10 Core RBAC Architectural Principles (Non-Negotiable)
  * 4.11 Document Pool Visibility & Permissions
    * 4.11.1 Visibility Rules by Creator
    * 4.11.2 Attribution & Badges
    * 4.11.3 Document Comments (Dispute Resolution)
    * 4.11.4 Audit Logging
* [SECTION 5 — AUTHENTICATION & ONBOARDING](application/auth-and-onboarding.md)
  * 5.1 Login Methods
    * Google OAuth Permission Scopes
  * 5.2 Session Persistence
  * 5.3 Super Admin Account Provisioning (Developer-Only)
  * 5.4 Editor Onboarding Flow
  * 5.5 Teacher & Organisation Onboarding Flow
  * 5.6 Sub-member Onboarding Flow
  * 5.7 Sub-member Leave Flow
* [SECTION 6 — UI DESIGN PRINCIPLES](application/ui-design-principles.md)
  * 6.1 Core Principles (Non-Negotiable)
  * 6.2 Design Handoff Protocol
* [SECTION 7 — APPLICATION ARCHITECTURE OVERVIEW](application/application-architecture.md)
  * 7.1 Three-Tier Platform Mental Model
  * 7.2 Core Architectural Rules
* [SECTION 8 — DASHBOARD SPECIFICATIONS](application/dashboard-specifications.md)
  * 8.0 Super Admin Dashboard
    * 8.0.1 Editor Management Panel
    * 8.0.2 Organisation Management Panel
    * 8.0.3 System-Wide Activity & Audit Log
    * 8.0.4 Platform-Level Metrics
    * 8.0.5 Super Admin Billing & Invoicing Dashboard
  * 8.1 Teacher Dashboard
    * 8.1.1 Entry Point Cards
    * 8.1.2 Kanban Board
    * 8.1.3 Logs Table
  * 8.2 Editor Dashboard
    * 8.2.0 Entry Point Cards
    * 8.2.1 Editor Kanban Board
    * 8.2.2 Editor Queue Management
    * 8.2.3 Priority Notification Panel
    * 8.2.4 Editor Document Metrics Dashboard
* [SECTION 9 — DOCUMENT TYPES & DATA MODELS](documents/types-and-data-models/)
  * 9.1 Document Type Overview
  * 9.2 Compilation Trigger Schedule
  * 9.3 Magazine Trigger Schedule
  * 9.3A Quarterly Collection Trigger Schedule
  * 9.3B Bi-Annual Compendium Trigger Schedule
  * 9.3C Annual Yearbook Trigger Schedule
  * 9.3D Category Extraction
  * 9.4 Newsletter Raw Data Structure (Markdown Hierarchy)
  * 9.5 Compilation & Magazine Raw Data Structure
  * 9.6 Document Status State Machine
* [SECTION 9A — PUBLICATION STREAMS (MULTI-STREAM DOCUMENT ARCHITECTURE)](documents/publication-streams.md)
  * 9A.1 Overview
  * 9A.2 Default Publication
  * 9A.3 Custom Publications
    * Who Can Create
    * Publication Approval Flow
    * Custom Publication Properties
  * 9A.4 Custom Document Types Within a Publication
    * Atomic Document Types (Mandatory — at least one)
    * Aggregated Document Types (Optional)
    * Naming Uniqueness Constraint
  * 9A.5 Mindmap Restrictions for Publications
  * 9A.6 Publication Management Access Matrix
  * 9A.7 Duplicate Document Prevention
    * Backend Logic
    * Duplicate Prevention Flow
    * UI Indicators
    * Version Control Alternative
  * 9A.8 Impact on Entry Point Cards
    * Teacher Dashboard Cards (Section 8.1.1)
    * Editor Dashboard Cards (Section 8.2.0)
  * 9A.9 Impact on Billing
  * 9A.10 Impact on Google Drive Folder Structure
* [SECTION 10 — NEWSLETTER CREATION FLOW (TEACHER — CANVAS EDITOR)](documents/types-and-data-models/newsletter/)
  * 10.1 Flow Initiation
  * 10.2 Metadata Sheet (Right Slide-in Panel)
  * 10.3 Block Editor Behaviour (Lexical.dev Canvas)
    * Adding Content
    * Field-Level Controls
    * Image Fields
    * Text Formatting Support
  * 10.4 Drag-and-Drop Reordering
  * 10.5 Sidebar Minimap (Table of Contents)
  * 10.6 Finalisation & Actions
    * Action 1 — Send to Editor
    * Action 2 — Export Document
    * Action 3 — Launch Presentation Mode
    * Action 4 — Translate Document
  * 10.7 Locked Document Behaviour
  * 10.8 Image Crop (Non-Destructive)
  * 10.9 Inline Pasted Reference Images
  * 10.10 Pre-Submission Q\&A Wizard
* [SECTION 11 — NEWSLETTER JSON SCHEMA (COMPLETE)](documents/types-and-data-models/newsletter/document-structure.md)
  * 11.1 Full Document JSON Structure
  * 11.2 Schema Notes for Developer
* [SECTION 11A — ATOMIC UID SYSTEM](documents/types-and-data-models/newsletter/atomic-uid-system.md)
  * 11A.1 Purpose
  * 11A.2 UID Format Specification
  * 11A.3 UID Generation Rules
  * 11A.4 Schema Integration
  * 11A.5 Image MIME-Level Metadata
  * 11A.6 UID in HTML Output
* [SECTION 13 — DOCUMENT PIPELINE — COMPLETE SLA STATE MACHINE](documents/types-and-data-models/newsletter/document-pipeline.md)
  * 13.1 Overview
  * 13.2 Pipeline Participants
  * 13.3 Stage-by-Stage Specification
    * STAGE 0 — Pre-Submission (Teacher's Domain)
    * STAGE 1 — Editor Raw Document Review (2-Hour SLA)
    * STAGE 2 — PDF Generation & Editor PDF Review (2-Hour SLA)
    * STAGE 3 — Delivery to Teacher
    * STAGE 4 — Revision Queue (Manual Only — No Auto-Continuation)
      * 13.3.4A Revision Screenshot Wizard (Mandatory)
  * 13.4 Real-Time Document Co-Visibility (Yjs CRDT)
  * 13.5 Non-Payment Pipeline Block
  * 13.6 Q\&A Late Addition — PDF Regeneration Flow
* [SECTION 14 — COMPILATION DOCUMENT WORKFLOW](documents/types-and-data-models/compilation.md)
  * 14.1 What a Compilation Is
  * 14.2 Trigger Mechanism
    * Automatic (Cron-Based)
    * Manual (Editor On-Demand Override)
  * 14.3 Newsletter Eligibility Rules
  * 14.4 Compilation Creation Pipeline
  * 14.5 Compilation File Naming Convention
* [SECTION 15 — MAGAZINE DOCUMENT WORKFLOW](documents/types-and-data-models/magazine.md)
  * 15.1 What a Magazine Is
  * 15.2 Trigger Mechanism
    * Automatic (Cron-Based)
    * Manual (Editor On-Demand Override)
  * 15.3 Magazine Creation Pipeline
  * 15.4 Magazine File Naming Convention
* [SECTION 15A — QUARTERLY COLLECTION WORKFLOW](documents/types-and-data-models/collection.md)
  * 15A.1 What a Quarterly Collection Is
  * 15A.2 Trigger Mechanism
    * Automatic (Cron-Based)
    * Manual (Editor On-Demand Override)
  * 15A.3 Quarterly Collection Creation Pipeline
  * 15A.4 Quarterly Collection File Naming Convention
* [SECTION 15B — BI-ANNUAL COMPENDIUM WORKFLOW](documents/types-and-data-models/compendium.md)
  * 15B.1 What a Bi-Annual Compendium Is
  * 15B.2 Trigger Mechanism
    * Automatic (Cron-Based)
    * Manual (Editor On-Demand Override)
  * 15B.3 Bi-Annual Compendium Creation Pipeline
  * 15B.4 Bi-Annual Compendium File Naming Convention
* [SECTION 15C — ANNUAL YEARBOOK WORKFLOW](documents/types-and-data-models/yearbook.md)
  * 15C.1 What an Annual Yearbook Is
  * 15C.2 Trigger Mechanism
    * Automatic (Cron-Based)
    * Manual (Editor On-Demand Override)
  * 15C.3 Annual Yearbook Creation Pipeline
  * 15C.4 Annual Yearbook File Naming Convention
* [SECTION 15D — CATEGORY EXTRACTION WORKFLOW](documents/types-and-data-models/extraction.md)
  * 15D.1 What a Category Extraction Is
  * 15D.2 Creation Flow (Editor-Only)
  * 15D.3 Category Extraction File Naming Convention
* [SECTION 16 — MINDMAP WORKFLOW](documents/types-and-data-models/mindmap.md)
  * 16.1 Overview
  * 16.2 Default Flow — Newsletter Pipeline (Optional Final Step)
  * 16.3 Standalone Flow — Independent Mindmap Creation
  * 16.4 Mind Elixir Technical Notes for Developer
  * 16.5 Mindmap in Translation Pipeline
* [SECTION 17 — EXPORT SYSTEM](documents/export-system.md)
  * 17.1 PPTX Export
  * 17.2 PDF Export via Reveal.js (PPTX → PDF pathway)
  * 17.3 TXT Export
  * 17.4 ZIP Export
  * 17.5 Reveal.js Presentation Mode (Live)
  * 17.6 PDF Pipeline Exports (Editor-Only — Branded WeasyPrint PDF)
  * 17.7 HTML Webpage Export (Editor-Only)
* [SECTION 18 — TRANSLATION SYSTEM](documents/translation-system.md)
  * 18.1 Overview
  * 18.2 Initiating Translation
  * 18.3 Translation Output
  * 18.4 Language Switcher
  * 18.5 Multilingual PPTX
  * 18.6 Multilingual PDF (Editor Pipeline)
  * 18.7 Per-Type Translation Toggle
* [SECTION 19 — TEMPLATE SYSTEM](documents/customisation/template-system.md)
  * 19.1 Overview
  * 19.2 Template Builder (Drag-and-Drop Micro App)
  * 19.3 Template Types & Use Cases
  * 19.4 Scope Controls (Per Template)
  * 19.5 Multiple Templates & Assignment
  * 19.6 Template Injection During PDF Generation
  * 19.7 Mindmap PDF Template Injection
* [SECTION 20 — AD BANNER MANAGEMENT SYSTEM](documents/customisation/ads-banner.md)
  * 20.1 Overview
  * 20.2 Ad Banner Configuration
  * 20.3 Document Type Assignment
  * 20.4 Ad Scheduling Controls
  * 20.5 Injection Logic
* [SECTION 21 — TEAM & SUB-MEMBER MANAGEMENT](organisation/team-management.md)
  * 21.1 Team Overview
  * 21.2 Invite Flow
  * 21.3 Sub-member Permission Configuration
  * 21.4 Sub-member Notification Settings
  * 21.5 Leave Organisation Flow
  * 21.6 Team Activity Log
* [SECTION 22 — NOTIFICATION SYSTEM](organisation/notification-system.md)
  * 22.1 Two-Tier Notification Architecture
  * 22.2 Complete Event-to-Notification Mapping
  * 22.3 User Notification Controls
* [SECTION 22A — CUSTOMER SUPPORT & TICKET SYSTEM](organisation/support-tickets.md)
  * 22A.1 Overview
  * 22A.2 Who Can Raise Tickets
  * 22A.3 Ticket Creation Flow
  * 22A.4 Ticket Lifecycle States
    * Auto-Close Policy
  * 22A.5 Ticket Communication Thread
  * 22A.6 Super Admin Ticket Dashboard
  * 22A.7 Ticket from Settings UI
  * 22A.8 Data Retention
* [SECTION 22B — OPENROUTER Q\&A SERVICE](organisation/openrouter-qa-service.md)
  * 22B.1 Overview
  * 22B.2 Model Resolution Chain
  * 22B.3 Request Lifecycle
  * 22B.4 Spend Guardrails
  * 22B.5 RBAC for Q\&A Actions
  * 22B.6 Translation Interaction
* [SECTION 23 — BILLING & MONETIZATION (RAZORPAY)](organisation/billing-and-monetization.md)
  * 23.1 Billing Model
  * 23.1A Discount Commitment Mechanism
    * How Discounts Activate
    * Commitment Period Tracking
    * Auto-Renewal
    * Refund Policy
  * 23.2 Razorpay Integration Architecture
  * 23.3 Billing Cycle Lifecycle
    * POSTPAID Billing (Default)
  * 23.3A Retrospective Document Billing Rules
    * 23.3A.1 What Makes a Document "Retrospective"
    * 23.3A.2 Backend Detection Logic
    * 23.3A.3 Retrospective Documents in Invoice Layout
    * 23.3A.4 UI Indicators for Retrospective Documents
    * 23.3A.5 Retrospective Documents from Previous Academic Year
    * PREPAID Billing (Optional)
  * 23.3B Document Billability Rules (is\_billable)
    * Default Behaviour
    * Editor Document Purpose Selection
    * Super Admin Override During Invoice Generation
  * 23.4 Manual Invoice Generation
  * 23.5 Invoice Archive
  * 23.6 Non-Payment Access Policy
  * 23.7 Super Admin Billing Dashboard Specification
  * 23.8 Evader Detection & Compliance Enforcement
    * 23.8.1 Problem Statement
    * 23.8.2 Existing Safeguards (Already in Place)
    * 23.8.3 Compliance Threshold
    * 23.8.4 Grace Period for New Organisations
    * 23.8.5 Post-Grace Enforcement Actions
      * Action 1: allow — Set Exception
      * Action 2: warning — Issue Compliance Warning
      * Action 3: temporarily-suspend — Suspend Organisation
      * Action 4: terminate — Permanently Terminate Organisation
    * 23.8.6 Super Admin Compliance Dashboard
* [SECTION 24 — DOCUMENT STORAGE ARCHITECTURE (4-LAYER)](api/document-storage.md)
  * 24.1 Overview
  * 24.2 Layer 1 — Browser Cache (Keystroke-Level)
  * 24.3 Layer 2 — Background Neon DB Sync (Every 30 Seconds)
  * 24.4 Layer 3 — Named Version on Finalisation
  * 24.5 Layer 4 — Google Drive Sync (Event-Triggered)
  * 24.6 Auto-Deletion & Data Retention Policy
* [SECTION 25 — GOOGLE DRIVE INTEGRATION](api/drive-api-integration.md)
  * 25.1 Drive Ownership Architecture
  * 25.2 Drive Folder Structure (Complete)
  * 25.3 Editor's CRUD Permissions on Drive
  * 25.4 Google Docs & HTML Mirror
* [SECTION 26 — CLOUDINARY & IMAGE LIFECYCLE MANAGEMENT](api/cloudinary-api-integration.md)
  * 26.1 Three-Tier Image Storage Architecture
  * 26.2 Image Upload Flow
  * 26.3 Image Naming Convention (Drive)
  * 26.4 Academic Year Boundary & Year-End Archival (Image-Only — Superseded by §26A when full-AY archive enabled)
  * 26.5 PDF Compression
  * 26.6 Image Storage Codec (WebP) & Render Transcoding
  * 26.7 File-Type Compression (Pre-Upload)
* [SECTION 26A — FULL-AY ARCHIVAL & RECOVERY](api/archival-and-recovery.md)
  * 26A.1 Overview & Toggle
  * 26A.2 Archive Cron Job
  * 26A.3 Archive ZIP Structure & Signed Manifest
  * 26A.4 Recovery Flow
  * 26A.5 Conflict Handling
  * 26A.6 Image Re-Upload on Recovery
* [SECTION 27 — YOUTUBE VIDEO INTEGRATION](api/youtube-api-integration.md)
  * 27.1 Connection
  * 27.2 Video Picker Flow — All Roles (Custom URL-Based UI)
  * 27.3 Video Picker Flow — Editor & Sub-member
  * 27.4 Video Link — Optional
  * 27.5 PDF Injection
* [SECTION 28 — ERROR HANDLING & RECOVERY LADDERS](api/error-handling-and-recovery.md)
  * 28.1 Image Upload Failure — Recovery Ladder
  * 28.2 Google Drive Sync Failure — Recovery
  * 28.3 WeasyPrint PDF Generation Failure — Recovery
  * 28.4 Storage Limit Failure — Drive & Neon DB
  * 28.5 Real-Time Connection Loss (WebSocket / Realtime)
  * 28.6 General API Error Handling
* [SECTION 29 — DATABASE SCHEMA OVERVIEW](database/schema-overview.md)
  * 29.1 Core Tables (Neon / PostgreSQL via Prisma)
    * super\_admins
    * organisations
    * editors
    * teachers
    * sub\_members
    * qa\_items
    * revision\_screenshots
    * openrouter\_config
    * ay\_archives
    * documents
    * document\_versions
    * document\_comments
    * pipeline\_events
    * pdf\_outputs
    * billing\_cycles
    * invoices
    * images
    * ad\_banners
    * templates
    * team\_activity\_logs
    * translations
    * audit\_logs
    * editor\_org\_assignments
    * atomic\_uid\_log
    * prepaid\_invoices
    * billing\_prepaid\_config
    * publications
    * publication\_document\_types
    * support\_tickets
    * ticket\_messages
    * ticket\_attachments
    * compliance\_records
    * discount\_commitments
* [SECTION 30 — DEPLOYMENT ARCHITECTURE](production/deployment-architecture.md)
  * 30.1 Infrastructure Stack
  * 30.2 Containerisation
  * 30.3 Scheduled Jobs (Google Cloud Scheduler)
  * 30.4 Real-Time Architecture Requirement
  * 30.5 Environment Configuration
* [SECTION 31 — NON-FUNCTIONAL REQUIREMENTS](production/non-functional-requirements.md)
  * 31.1 Performance
  * 31.2 Security
  * 31.3 Accessibility
  * 31.4 Scalability
  * 31.5 Monitoring (Pre-Deployment)
  * 31.6 PDF Compression
  * 31.7 Browser Support
* [SECTION 32 — DEVELOPER HANDOFF NOTES & DESIGN ASSETS](misc/developer-handoff-notes.md)
  * 32.1 Design Assets
  * 32.2 Mandatory Component Choices
  * 32.3 Key Implementation Dependencies (Developer Must Research)
  * 32.4 JSON5 Comment Handling
  * 32.6 Build Order Recommendation
  * 32.7 Definition of Done (Per Feature)
* [APPENDICES](misc/appendices.md)
  * APPENDIX A — IMAGE NAMING CONVENTION REFERENCE
  * APPENDIX B — DOCUMENT ID FORMATS
  * APPENDIX C — FILE NAMING CONVENTIONS SUMMARY
  * APPENDIX D — AGGREGATION TRIGGER SCHEDULE QUICK REFERENCE
  * APPENDIX E — BILLING QUICK REFERENCE
