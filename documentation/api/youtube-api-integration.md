---
icon: youtube
---

# Youtube Video Integration

## SECTION 27 — YOUTUBE VIDEO INTEGRATION

> **Scope notice**: The YouTube Data API OAuth scope is **not used** anywhere in this application. No role — including Teacher — requires or requests YouTube API credentials. All YouTube video discovery and linking is performed via a custom URL-based interface backed by a server-side public channel feed fetch. This keeps the OAuth consent screen limited to the approved non-sensitive scopes (Drive + profile only).

### 27.1 Connection

* Each Teacher manually configures their YouTube channel reference in Account Settings → Integrations → YouTube.
* No OAuth handshake with YouTube is performed — the Teacher simply provides their `channel_ID` and/or `channel_URL` as plain text.
* The connection is per-Teacher and is stored in the Teacher's organisation settings record.
* No YouTube API credentials, tokens, or read scopes are requested at any point in the authentication or onboarding flow.

### 27.2 Video Picker Flow — All Roles (Custom URL-Based UI)

All roles — Teacher, Editor, and Sub-member — use the **same custom URL-based video picker UI**. There is no API-authenticated flow for any role.

1. The user opens the Metadata Sheet (right slide-in panel during document creation or review).
2. The user clicks the YouTube video field or "Browse Channel" button.
3. The system reads the Teacher's configured `channel_ID` / `channel_URL` from the organisation settings.
4. The backend (FastAPI) attempts to fetch the channel's public video listing server-side:
   * **Primary path**: Server-side HTTP fetch of the public channel feed using the Teacher's `channel_ID` or `channel_URL` — no YouTube API key required for public channel listings.
   * **Fallback path**: If the server-side fetch fails (e.g. rate-limited or blocked), the UI falls back to rendering an embedded iframe of `https://www.youtube.com/channel/{channel_ID}/videos` so the user can still browse and manually provide the URL.
5. A modal renders the channel's video list with thumbnail, title, upload date, and duration.
6. The user selects a video — its URL is populated in the `video-url` field of the document metadata.
7. The backend optionally fetches supplemental video metadata (title, thumbnail, duration, channel name) via a lightweight public URL parse and stores it in `metadata.video-metadata`; no YouTube API key is required for this.

**Implementation note**: The backend caches the channel feed response for a configurable TTL (default: 1 hour) to avoid excessive requests. The iframe fallback renders inside the same modal as a full-page channel browse view — the user can copy the video URL directly from the iframe if the primary list fails to load.

### 27.3 Video Picker Flow — Editor & Sub-member

Editors and Sub-members use the **identical flow** described in Section 27.2. There is no distinction between the Teacher flow and the Editor/Sub-member flow — all roles use the custom URL-based interface backed by the same FastAPI server-side fetch + iframe fallback mechanism.

### 27.4 Video Link — Optional

* The YouTube video link is **optional** for every Newsletter.
* Any role can skip the video picker and leave `video-url = null`.
* Teacher can attach or update the YouTube video link at any point before the document is finalised.
* Editor can also attach or update the YouTube video link on the Teacher's behalf during the Editor's Stage 1 review.
* Sub-member can also attach or update the YouTube video link via the custom URL-based UI (Section 27.2).

### 27.5 PDF Injection

* The YouTube video URL is dynamically injected into the PDF during WeasyPrint rendering.
* Injection point: wherever the organisation's configured template places the `{yt_url}` dynamic placeholder block.
* If no template is configured, or the template does not include a `{yt_url}` block, the URL is not embedded in the PDF.
* The URL in the PDF is rendered as a clickable hyperlink (supported by WeasyPrint's PDF link generation).
