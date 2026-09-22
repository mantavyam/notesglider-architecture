---
icon: microchip
metaLinks:
  alternates:
    - /broken/spaces/yE16Xb3IemPxJWydtPOj/pages/QPzbTvC6XsT5gERiU43E
---

# Technology Stack

## SECTION 3 — TECHNOLOGY STACK

### 3.1 Complete Stack

| Layer                    | Technology                                                                                                               | Purpose                                                                                                                                                                              |
| ------------------------ | ------------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **Frontend**             | Next.js (App Router)                                                                                                     | Primary web application framework                                                                                                                                                    |
| **UI Components**        | ShadCN UI + compatible third-party ShadCN registries (e.g. ui.tripled.work, or as discovered via shoogle.dev MCP server) | Core component library — mandatory. Use direct ShadCN or pre-made components from compatible third-party ShadCN registries                                                           |
| **Rich Text Editor**     | Lexical.dev                                                                                                              | Block-based document canvas editor                                                                                                                                                   |
| **Mind Map Canvas**      | Mind Elixir Core (`mind-elixir-core`)                                                                                    | Interactive mind map editor                                                                                                                                                          |
| **Backend**              | Python + FastAPI                                                                                                         | API server, document processing, pipeline orchestration                                                                                                                              |
| **PDF Generation**       | WeasyPrint                                                                                                               | HTML-to-PDF conversion engine                                                                                                                                                        |
| **Slide Presentation**   | Reveal.js                                                                                                                | Browser-based presentation mode                                                                                                                                                      |
| **PPTX Generation**      | **PptxGenJS** (client-side, browser/Node JS) — `https://github.com/gitbrent/PptxGenJS`                                   | Client-side `.pptx` file generation from structured JSON. Replaces v4 `python-pptx`. Built-in table auto-pagination; text pagination via shared height-estimator utility. See §17.1. |
| **AI Q\&A Generation**   | OpenRouter API (Structured Outputs)                                                                                      | Q\&A item generation per §10.10 / §22B. Platform-wide API key; Super-Admin-configurable model + 4 fallbacks; per-Org token cap.                                                      |
| **Database**             | Neon (PostgreSQL, accessed via Prisma ORM)                                                                               | Primary data store                                                                                                                                                                   |
| **Image CDN**            | Cloudinary                                                                                                               | Active-year image delivery and storage                                                                                                                                               |
| **Google OAuth**         | Better-Auth (with Google Identity provider)                                                                              | Primary authentication (Google OAuth, Magic Link, Email+Password)                                                                                                                    |
| **Google Drive API**     | Google Drive                                                                                                             | Per-user cloud file sync and archival                                                                                                                                                |
| **Google Docs API**      | Google Docs                                                                                                              | Parallel `.gdoc` save for online preview                                                                                                                                             |
| **Google Translate API** | Cloud Translation                                                                                                        | Multilingual document generation                                                                                                                                                     |
| **YouTube (No API Key)** | Public channel URL fetch (server-side) + iframe fallback                                                                 | Channel feed browsing for video link attachment — no YouTube Data API OAuth scope required                                                                                           |
| **Payments**             | Razorpay API                                                                                                             | Super Admin-managed master Razorpay account for platform-wide billing and invoice management                                                                                         |
| **Email Automation**     | Resend                                                                                                                   | Notifications and invoice delivery                                                                                                                                                   |
| **Deployment**           | Google Cloud Run (containerised, serverless)                                                                             | Frontend and backend deployment                                                                                                                                                      |
| **Scheduling**           | Google Cloud Scheduler                                                                                                   | Cron jobs for Compilation triggers, billing cycles, archival                                                                                                                         |

### 3.2 Key Library References

#### Frontend

{% embed url="https://nextjs.org/" %}

**Component Library:**

{% embed url="https://ui.shadcn.com/" %}

**Preferred Shadcn Compatible Styled Component Library:**

{% embed url="https://ui.tripled.work/components" %}

**MCP Server to Search all 3rd party Shadcn Compatible Directories:**

{% embed url="https://shoogle.dev/mcp-install" %}

#### Backend

{% embed url="https://www.python.org/" %}

#### APIs

{% embed url="https://fastapi.tiangolo.com/" %}

#### Authentication

**Better-Auth — Framework-agnostic authentication library:**

{% embed url="https://www.better-auth.com/" %}

* Supports Google OAuth, Magic Link, and Email+Password authentication
* Prisma-compatible database adapter
* Session management with HttpOnly cookies
* TypeScript-first with full type safety

{% embed url="https://cloud.google.com/" %}

**Google Services:**

{% embed url="https://docs.cloud.google.com/translate/docs/translate-text" %}

***

<details>

<summary><strong>Google Drive:</strong> Use these REST APIs to interact programmatically with Drive.</summary>

{% embed url="https://developers.google.com/workspace/drive/api" %}

* Upload, download, share, and manage files stored in Google Drive.<a href="https://developers.google.com/workspace/drive/api" class="button secondary">View documentation</a>

{% embed url="https://developers.google.com/workspace/drive/activity" %}

* Get info about user activity on files and folders.<a href="https://developers.google.com/workspace/drive/activity" class="button secondary">View documentation</a>

{% embed url="https://developers.google.com/workspace/drive/labels" %}

* Apply and manage labels on your Drive files and folders, and search for files using metadata terms defined by a custom label taxonomy.<a href="https://developers.google.com/workspace/drive/labels" class="button secondary">View documentation</a>

{% embed url="https://developers.google.com/workspace/drive/picker" %}

* Embed a file manager widget in your web app.<a href="https://developers.google.com/workspace/drive/picker" class="button secondary">View documentation</a>

{% embed url="https://developers.google.com/workspace/docs/api/how-tos/overview" %}

</details>

***

Access and update Google Slides programmatically with popular programming languages, including Java, JavaScript, and Python.<a href="https://developers.google.com/workspace/slides/api/guides/overview" class="button secondary">View documentation</a>

{% embed url="https://developers.google.com/workspace/slides/api/guides/overview" %}

***

Access and update Google Docs programmatically, just like any other user.<a href="https://developers.google.com/workspace/docs/api/how-tos/overview" class="button secondary">View documentation</a>

{% embed url="https://developers.google.com/workspace/docs/api/how-tos/overview" %}

***

YouTube has a number of APIs and tools that let you embed YouTube functionality into your own website and applications.

{% embed url="https://developers.google.com/youtube/documentation" %}

#### Utility

**Rich Text Editor:**

{% embed url="https://lexical.dev/" %}

**Dynamic Slides Generation:**

{% embed url="https://revealjs.com/" %}

**Dynamic Document Generation:**

{% embed url="https://weasyprint.org/" %}

**Dynamic Mindmap Generation:**

{% embed url="https://github.com/SSShooter/mind-elixir-core" %}

{% embed url="https://docs.mind-elixir.com/" %}

**Email:**

{% embed url="https://resend.com/docs/introduction" %}

**PPTX Generation (v5 — client-side JS):**

{% embed url="https://github.com/gitbrent/PptxGenJS" %}

<details>

<summary><strong>Other Notable Mentions (non-PPTX):</strong></summary>

{% embed url="https://pypi.org/project/pypandoc/" %}

{% embed url="https://pandoc.org/" %}

</details>

#### Database

**Primary Storage**

{% embed url="https://neon.com/" %}

**For Image Storage CDN:**

{% embed url="https://cloud.google.com/storage" %}

or

{% embed url="https://cloudinary.com/" %}

#### Payments

{% embed url="https://razorpay.com/docs/api/?preferred-country=IN" %}

#### Hosting

{% embed url="https://cloud.google.com/run" %}

#### Scheduler

{% embed url="https://docs.cloud.google.com/scheduler/docs/schedule-run-cron-job?_gl=1*9ms8py*_ga*MTA0ODE5NzQxNC4xNzcxNzUwOTA4*_ga_WH2QY8WWF5*czE3NzI2MTQ2MTckbzgkZzEkdDE3NzI2MTQ2MzAkajQ3JGwwJGgw" %}
