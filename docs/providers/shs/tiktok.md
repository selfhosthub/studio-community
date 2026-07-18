# TikTok Provider

Publish videos and photos, manage comments, and retrieve analytics from TikTok.

## Authentication

TikTok uses **OAuth2** for authentication. You can connect in two ways:

| Method | Who sets it up | User experience |
|--------|---------------|-----------------|
| **Platform OAuth** (recommended) | Operator sets `TIKTOK_CLIENT_ID` and `TIKTOK_CLIENT_SECRET` env vars once | Users click "Connect with TikTok" - no developer app needed |
| **Org-managed OAuth** | Each organization creates their own TikTok developer app | Users enter their own client key/secret, then authorize |

### Setup Overview

```
+---------------------------+       +----------------------------+
|   TikTok Developer Portal |       |      Studio (.env)         |
+---------------------------+       +----------------------------+
| 1. Create App             |       | TIKTOK_CLIENT_ID=xxx       |
| 2. Add Products:          |       | TIKTOK_CLIENT_SECRET=xxx   |
|    - Login Kit            |       | API_BASE_URL=https://...   |
|    - Content Posting API  |       | FRONTEND_URL=https://...   |
| 3. Add scopes             |       +----------------------------+
| 4. Set Redirect URI:      |                   |
|    {API_BASE_URL}/api/v1/ |                   v
|    oauth/tiktok/callback  |       +----------------------------+
| 5. Verify domain          |       |   Studio UI                |
+---------------------------+       |   Providers > TikTok >     |
                                    |   Click "Authorize"        |
                                    +----------------------------+
```

### OAuth Authorization Flow

```
Browser                    Studio API                 TikTok
   |                          |                          |
   |-- Click "Authorize" ---->|                          |
   |                          |-- Generate PKCE -------->|
   |                          |   (code_verifier +       |
   |                          |    code_challenge)       |
   |                          |-- Store state in Redis   |
   |                          |                          |
   |<-- Redirect to TikTok ---|                          |
   |                          |                          |
   |-- User grants consent --------------------------->  |
   |                                                     |
   |<-- Redirect to callback with ?code=xxx&state=xxx ---|
   |                          |                          |
   |-- GET /callback -------->|                          |
   |                          |-- Verify state (Redis)   |
   |                          |-- Exchange code -------->|
   |                          |   (+ code_verifier)      |
   |                          |<-- access_token ---------|
   |                          |    refresh_token         |
   |                          |-- Store tokens (DB) ---->|
   |                          |                          |
   |<-- Redirect to UI -------|                          |
   |   /providers?oauth_success=true                     |
```

---

## Prerequisites

Before starting, you need:

- A TikTok account
- An approved [TikTok Developer App](https://developers.tiktok.com/apps/)
- Your app must have the required scopes approved (see below)

> **Developer app approval:** TikTok requires app review before granting API access. Apply early - approval can take several days.

---

## Step 1: Create a TikTok Developer App

1. Go to [TikTok for Developers](https://developers.tiktok.com/)
2. Sign in and go to **My Apps**
3. Click **Create App** (or **Connect an app**)
4. Fill in app details:
   - **App Name:** Self-Host Studio (or your white-label name)
   - **Description:** Workflow automation platform for content publishing, comment management, and analytics
   - **Terms of Service URL:** `https://your-domain.com/terms.html`
   - **Privacy Policy URL:** `https://your-domain.com/privacy.html`

### Add Products

TikTok uses **products** (not standalone scopes). Add these products to your app - each product brings its own scopes:

| Product | Scopes it adds | Required for |
|---------|---------------|-------------|
| **Login Kit** | `user.info.basic` | OAuth login, Get User Profile, Get Creator Info |
| **Content Posting API** | `video.publish`, `video.upload` | Publish Video, Publish Photo, Check Status, Cancel |

Additionally, add these **scopes** via **+ Add scopes**:

| Scope | Required for |
|-------|-------------|
| `user.info.profile` | Bio, profile links, verification status |
| `user.info.stats` | Follower, following, likes, and video counts |
| `video.list` | List Videos, Get Video Details, List Comments, Reply to Comment |

Within **Content Posting API**, enable the **Direct Post** setting and **verify your domain** (required for pull-from-URL publishing).

### Domain Verification

TikTok requires you to verify ownership of the domain used for pull-from-URL publishing. This involves adding DNS records:

1. In the TikTok developer portal, go to your app's **Content Posting API** settings
2. Click **Verify Domain** and copy the verification string TikTok provides
3. In your DNS provider (e.g., Cloudflare), add a **TXT record**:
   - **Name:** your domain (e.g., `app.yourdomain.com`)
   - **Content:** the verification string from TikTok
4. If using Cloudflare Tunnel, also confirm the **CNAME record** exists for your tunnel subdomain (Cloudflare usually creates this automatically when you add a public hostname)
5. Back in TikTok, click **Verify** - TikTok checks the TXT record

> DNS propagation can take a few minutes. If verification fails, wait and retry.

> **Product approval:** Each product must be reviewed by TikTok. You can start with **Login Kit** only and add Content Posting API later.
>
> **Unaudited apps** can publish content but it will be restricted to **private viewing mode** and limited to **5 users per 24 hours**. Pass TikTok's audit to lift these restrictions.

---

## Step 2: Configure Redirect URI

In your TikTok developer app settings, add the redirect URI:

```
https://your-studio-domain.com/api/v1/oauth/tiktok/callback
```

For local development:
```
http://localhost:8000/api/v1/oauth/tiktok/callback
```

> **Redirect URI must match exactly.** TikTok is strict about trailing slashes and protocol.

---

## Step 3: Connect in Studio

### Option A: Platform OAuth (Recommended)

The operator sets environment variables once. All users can connect with one click.

**1. Set environment variables** (in the API container's env):
```env
TIKTOK_CLIENT_ID=your_client_key
TIKTOK_CLIENT_SECRET=your_client_secret
API_BASE_URL=https://your-studio-domain.com
FRONTEND_URL=https://your-studio-domain.com
```

`API_BASE_URL` is required so the OAuth redirect URI matches what you registered in the TikTok developer portal. `FRONTEND_URL` ensures the post-authorization redirect returns to the correct domain.

> **Local development with Cloudflare Tunnel:** If you're exposing a local instance via `cloudflared` (e.g., for TikTok app review), set `API_BASE_URL` and `FRONTEND_URL` to your tunnel domain (e.g., `https://app.your-domain.com`). The redirect URI registered in TikTok must match `API_BASE_URL` exactly.

**2. Restart the API** to pick up the new env vars.

**3. Connect:**
1. Go to **Providers > TikTok > Credentials**
2. Click **Connect with TikTok**
3. Authorize in the TikTok login flow
4. The credential shows **Connected** status

### Option B: Org-Managed OAuth

Each organization provides their own TikTok developer app credentials.

1. Go to **Providers > TikTok > Credentials**
2. Click **Add Credential**
3. Enter your **Client Key** and **Client Secret**
4. Save the credential
5. Click **Authorize** on the credential
6. Sign in with TikTok and grant permissions
7. The credential shows **Connected** status

---

## Available Services

### Creator & Profile

| Service | Description |
|---------|-------------|
| **Get Creator Info** (`tiktok_get_creator_info`) | Check posting permissions, available privacy levels, and max video duration before publishing |
| **Get User Profile** (`tiktok_get_user_info`) | Retrieve display name, avatar, follower/following/likes/video counts, and bio |

### Video Management

| Service | Description |
|---------|-------------|
| **List My Videos** (`tiktok_list_videos`) | Fetch recent videos with engagement metrics (views, likes, comments, shares). Paginated, max 20 per request |
| **Get Video Details** (`tiktok_get_video_info`) | Retrieve detailed stats for specific videos by ID (up to 20 at once) |

### Comments

| Service | Description |
|---------|-------------|
| **List Video Comments** (`tiktok_list_comments`) | Get comments on a video with pagination (max 50 per request) |
| **Reply to Comment** (`tiktok_reply_to_comment`) | Post a reply to an existing comment (max 150 characters) |

### Publishing

| Service | Description |
|---------|-------------|
| **Publish Video** (`tiktok_publish_video`) | Publish a video via pull-from-URL - TikTok downloads the video from a public URL |
| **Publish Photo Post** (`tiktok_publish_photo`) | Publish a photo carousel with up to 35 images via pull-from-URL |
| **Check Publish Status** (`tiktok_check_publish_status`) | Poll publish status - returns `PROCESSING_DOWNLOAD`, `PUBLISH_COMPLETE`, or `FAILED` |
| **Cancel Publish** (`tiktok_cancel_publish`) | Best-effort cancel of an in-progress publish operation |

### Inactive Services (Phase 2)

These require a custom adapter enhancement for chunked file upload:

| Service | Description |
|---------|-------------|
| **Init Video Upload** (`tiktok_init_video_upload`) | Initialize file-based video upload for non-public URLs |
| **Upload Video Chunk** (`tiktok_upload_video_chunk`) | Upload video file chunks to the upload URL |

---

## Publishing Workflow

TikTok video publishing uses a **pull-from-URL** model - you provide a publicly accessible video URL and TikTok downloads it.

### Typical Video Publish Workflow

```
 +-----------------------+
 | 1. Generate Content   |     shs-comfyui, OpenAI, etc.
 |    (AI / Creative)    |
 +-----------+-----------+
             |
             v
 +-----------------------+
 | 2. Create Video       |     shs_create_video
 |    (Local GPU)        |---> storage_url (public)
 +-----------+-----------+
             |
             v
 +-----------------------+
 | 3. Get Creator Info   |     tiktok_get_creator_info
 |    (Check Limits)     |---> privacy_levels, max_duration
 +-----------+-----------+
             |
             v
 +-----------------------+
 | 4. Publish Video      |     tiktok_publish_video
 |    (Pull from URL)    |---> publish_id
 +-----------+-----------+     TikTok downloads the video
             |
             v
 +-----------------------+
 | 5. Check Status       |     tiktok_check_publish_status
 |    (Poll)             |---> PUBLISH_COMPLETE or FAILED
 +-----------------------+
```

### Privacy Levels

| Level | Who can see |
|-------|-------------|
| `PUBLIC_TO_EVERYONE` | Anyone on TikTok |
| `MUTUAL_FOLLOW_FRIENDS` | Only mutual followers |
| `FOLLOWER_OF_CREATOR` | Only your followers |
| `SELF_ONLY` | Only you (recommended for testing) |

> **Always use `SELF_ONLY` for testing.** Call `get_creator_info` first to check which privacy levels your account supports.

### Video Requirements

- **Format:** MP4 or WebM
- **Duration:** 3 seconds to 10 minutes (check `max_video_post_duration_sec` from creator info)
- **Max size:** 4 GB
- **URL must be publicly accessible** - TikTok's servers download the file

### Photo Post Requirements

- **Max images:** 35 per post
- **Image URLs must be publicly accessible**
- **Title:** max 90 characters
- **Description:** max 4000 characters

---

## Token Management

- **Access tokens** expire after a short period
- **Refresh tokens** are used automatically to get new access tokens
- Workers refresh tokens automatically before making API calls

**When refresh fails:**
- The user may have revoked app access in TikTok settings
- Click **Re-authorize** to get fresh tokens

---

## Troubleshooting

### Common Errors

| Error | Cause | Solution |
|-------|-------|----------|
| **`spam_risk_too_many_posts`** | Posting too frequently | Wait and try again later |
| **`spam_risk_user_banned_from_posting`** | Account restricted | Check TikTok app for account status |
| **`publish_status: FAILED`** | Video didn't meet requirements | Check `fail_reason` in status response |
| **`privacy_level_option_mismatch`** | Requested privacy level not available | Call `get_creator_info` to check available levels |
| **401 Unauthorized** | Token expired or revoked | Click **Refresh** or **Re-authorize** |
| **`url_ownership_unverified`** | Video URL not accessible by TikTok | Ensure URL is publicly accessible (no auth required) |
| **`error_type=redirect_uri`** | `API_BASE_URL` not set or doesn't match registered redirect URI | Set `API_BASE_URL` env var to match your registered domain |
| **`error_type=code_challenge`** | PKCE not supported by your Studio version | Update to latest Studio - PKCE is now always enabled |

### Publish Status Not Completing

1. Publishing is async - TikTok downloads the video after you call publish
2. Poll `check_publish_status` - typical flow is `PROCESSING_DOWNLOAD` → `PUBLISH_COMPLETE`
3. If stuck in `PROCESSING_DOWNLOAD`, the video URL may be slow or unreachable from TikTok's servers
4. If `FAILED`, check the `fail_reason` field for details

### OAuth Flow Not Working

**Re-authorize button missing:**
- Verify `TIKTOK_CLIENT_ID` and `TIKTOK_CLIENT_SECRET` are set on the **API container**
- Restart the API after adding env vars

**Redirect URI mismatch (`error_type=redirect_uri`):**
- Set `API_BASE_URL` to your public domain (e.g., `https://app.your-domain.com`)
- The redirect URI sent to TikTok is `{API_BASE_URL}/api/v1/oauth/tiktok/callback`
- This must exactly match what you registered in TikTok developer settings
- **No trailing slash** - TikTok is strict about exact match
- Common mistake: leaving `API_BASE_URL` unset (defaults to `http://localhost:8000`)

**Redirects to login after authorization:**
- Set `FRONTEND_URL` to the same domain you access Studio from
- After TikTok authorization, the callback redirects to `{FRONTEND_URL}/providers/...`
- If `FRONTEND_URL` doesn't match your browser session domain, you'll land on the login page

---

## Rate Limits

TikTok enforces rate limits per app and per user. Key limits:

- **Publishing:** Limited number of posts per day (varies by account)
- **Read operations:** Generally generous but subject to change
- Check [TikTok API documentation](https://developers.tiktok.com/doc/overview) for current limits

---

## Security Notes

- OAuth credentials are stored encrypted in the organization's credential store
- Platform OAuth credentials (env vars) are never exposed to users
- Each organization has completely isolated credentials
- Access tokens are refreshed automatically - refresh tokens are stored securely

## Terms

Your use of TikTok is governed by TikTok's own terms, not by Studio's: [https://www.tiktok.com/legal/page/global/tik-tok-developer-terms-of-service/en](https://www.tiktok.com/legal/page/global/tik-tok-developer-terms-of-service/en). Costs, rate limits, content-ownership rules, and acceptable-use policies are set by the provider. You are responsible for complying with them and for any charges you incur. See LEGAL.md in the Studio repository.
