# YouTube Provider

Upload, manage, and search YouTube videos, playlists, and comments.

## Authentication

YouTube uses **Google OAuth2** for authentication. You can connect in two ways:

| Method | Who sets it up | User experience |
|--------|---------------|-----------------|
| **Platform OAuth** (recommended) | Operator sets `GOOGLE_CLIENT_ID` and `GOOGLE_CLIENT_SECRET` env vars once | Users click "Connect with Google" — no project needed |
| **Org-managed OAuth** | Each organization creates their own Google Cloud project | Users enter their own client ID/secret, then authorize |

Both methods use the same Google Cloud setup. The difference is who provides the OAuth credentials.

---

## Prerequisites

Before starting, you need:

- A Google account (personal or Workspace)
- A YouTube channel linked to that account
- Access to [Google Cloud Console](https://console.cloud.google.com/)

> **Brand accounts:** YouTube brand accounts work for API uploads. If you get 403 errors, verify the account has upload permissions (see [Troubleshooting](#troubleshooting)).

---

## Step 1: Create a Google Cloud Project

1. Go to [Google Cloud Console](https://console.cloud.google.com/)
2. Click **Select a project** → **New Project**
3. Name it (e.g., "Studio YouTube") and click **Create**
4. Select the new project

### Enable the YouTube Data API

1. Go to **APIs & Services > Library**
2. Search for **YouTube Data API v3**
3. Click **Enable**

> **Important:** You must enable **YouTube Data API v3** specifically. Other YouTube APIs (Analytics, Reporting) are separate and not needed for basic operations.

---

## Step 2: Configure OAuth Consent Screen

1. Go to **APIs & Services > OAuth consent screen**
2. Choose **External** (or **Internal** if using Google Workspace)
3. Fill in required fields:
   - **App name:** Your app name (e.g., "Studio")
   - **User support email:** Your email
   - **Developer contact:** Your email
4. Click **Save and Continue**

### Add Scopes

1. Click **Add or Remove Scopes**
2. Search for and add these scopes:
   - `https://www.googleapis.com/auth/youtube` — Manage your YouTube account
   - `https://www.googleapis.com/auth/youtube.upload` — Upload videos
   - `https://www.googleapis.com/auth/youtube.readonly` — View account info
   - `https://www.googleapis.com/auth/youtube.force-ssl` — Manage videos (required for comments, playlists)
3. Click **Update** → **Save and Continue**

### Add Test Users (Testing Mode)

If your app is in **Testing** status (not verified):

1. Click **Add Users**
2. Add the Gmail address of every Google account that will authorize
3. Click **Save and Continue**

> **Testing mode limits:** Only added test users can authorize. Tokens expire after 7 days. For production use, submit for verification.

---

## Step 3: Create OAuth Credentials

1. Go to **APIs & Services > Credentials**
2. Click **Create Credentials > OAuth client ID**
3. Application type: **Web application**
4. Name: "Studio" (or any name)
5. Under **Authorized redirect URIs**, add:
   ```
   https://your-studio-domain.com/api/v1/oauth/google/callback
   ```
   For local development:
   ```
   http://localhost:8000/api/v1/oauth/google/callback
   ```
6. Click **Create**
7. Copy the **Client ID** and **Client Secret**

> **Redirect URI must match exactly.** If you get "redirect_uri_mismatch" errors, check for trailing slashes, http vs https, and port numbers.

---

## Step 4: Connect in Studio

### Option A: Platform OAuth (Recommended)

The operator sets environment variables once. All users can connect with one click.

**1. Set environment variables** (in the API container's env):
```env
GOOGLE_CLIENT_ID=123456789-abc.apps.googleusercontent.com
GOOGLE_CLIENT_SECRET=GOCSPX-xxxxxxxxxxxxx
```

**2. Restart the API** to pick up the new env vars.

**3. Connect:**
1. Go to **Providers > YouTube > Credentials**
2. Click **Connect with Google**
3. Select your Google account in the account picker
4. Grant YouTube permissions
5. The credential shows **Connected** status

**Re-authorize:** To switch Google accounts or update scopes, click **Re-authorize** on the credential.

### Option B: Org-Managed OAuth

Each organization provides their own Google credentials.

1. Go to **Providers > YouTube > Credentials**
2. Click **Add Credential**
3. Enter your **Client ID** and **Client Secret** from Step 3
4. Save the credential
5. Click **Authorize** on the credential
6. Sign in with your Google account and grant permissions
7. The credential shows **Connected** status

---

## Available Services

### Video Management

| Service | Description |
|---------|-------------|
| **List Videos** | Get videos from the authenticated user's channel |
| **Get Video** | Retrieve detailed video information by ID |
| **Search** | Search YouTube for videos, channels, or playlists |
| **Update Video** | Update video title, description, tags, or privacy |
| **Delete Video** | Delete a video from your channel |

### Video Upload (Resumable)

YouTube uses a **two-step resumable upload** process:

| Step | Service | What it does |
|------|---------|--------------|
| 1 | **Init Upload** | Creates an upload session, returns an upload URL |
| 2 | **Upload Bytes** | Streams the video file to the upload URL |

**How it works in a workflow:**

```
Step 1: Init Upload
  → Sends video metadata (title, description, privacy)
  → Returns upload_url in response headers (Location header)

Step 2: Upload Bytes
  → Uses {{ step1.response_headers.location }} as the endpoint
  → Streams the video file from storage
  → Returns the created video resource
```

The Init Upload step sends metadata as JSON with `uploadType=resumable` and `part=snippet,status` as query parameters. The Upload Bytes step streams raw bytes with zero query parameters (the upload URL already contains all needed parameters).

### Thumbnails

| Service | Description |
|---------|-------------|
| **Set Thumbnail** | Upload a custom thumbnail image for a video |

### Playlists

| Service | Description |
|---------|-------------|
| **List Playlists** | Get playlists owned by the authenticated user |
| **Create Playlist** | Create a new playlist |
| **Add to Playlist** | Add a video to a playlist |
| **Remove from Playlist** | Remove an item from a playlist |

### Comments

| Service | Description |
|---------|-------------|
| **List Comments** | Get comments on a video (with threading) |
| **Post Comment** | Post a top-level comment on a video |
| **Reply to Comment** | Reply to an existing comment |

### Channel Info

| Service | Description |
|---------|-------------|
| **List Channels** | Get channel details, statistics, and branding |

---

## Token Management

- **Access tokens** expire after **1 hour**
- **Refresh tokens** are used automatically to get new access tokens
- Workers refresh tokens automatically before making API calls

**Manual refresh:** Click the **Refresh** button on the credential card, or:
```
POST /api/v1/oauth/google/refresh/{credential_id}
```

**When refresh fails:**
- The refresh token may have been revoked (user removed app access in Google Account settings)
- Click **Re-authorize** to get fresh tokens

**Testing mode token expiry:** In Google Cloud testing mode, tokens expire after 7 days. You'll need to re-authorize weekly until your app is verified.

---

## Troubleshooting

### Common Errors

| Error | Cause | Solution |
|-------|-------|----------|
| **403 Forbidden** `youtube.video` | Account lacks upload permission, or bad request parameters | See [403 Debugging](#403-forbidden-on-upload) below |
| **401 Unauthenticated** | Access token expired or invalid | Click **Refresh** or **Re-authorize** |
| **400 Bad Request** `uploadType` | Missing query parameters on init upload | Verify `parameterMapping.query` includes `uploadType` and `part` |
| **redirect_uri_mismatch** | OAuth callback URL doesn't match Google Console | Add the exact redirect URI shown in the error |
| **access_denied** | User denied permissions or not in test users list | Check OAuth consent screen test users |
| **Token refresh failed** | Refresh token revoked or expired | Click **Re-authorize** |
| **"Provider does not support OAuth"** | Provider config missing `oauth` section | Rebuild DB to reinstall provider with OAuth config |
| **No Re-authorize button** | Platform OAuth not detected | Check env vars are set and API is restarted |

### 403 Forbidden on Upload

The most common upload error. Debug in this order:

**1. Verify the API is enabled:**
- Go to [Google Cloud Console](https://console.cloud.google.com/) → **APIs & Services > Enabled APIs**
- Confirm **YouTube Data API v3** shows as enabled

**2. Check OAuth scopes:**
- The credential must have `youtube.upload` scope
- Go to **Providers > YouTube > Credentials**
- If the scope was added after initial authorization, click **Re-authorize**

**3. Verify the account has a channel:**
- Go to [youtube.com](https://youtube.com) and confirm you can upload manually
- Some Google Workspace accounts don't have YouTube channels by default

**4. Test the token directly with curl:**
```bash
# Get the access token (from worker internal endpoint)
curl -s -H "X-Worker-Secret: $WORKER_SHARED_SECRET" \
  "http://localhost:8000/api/v1/internal/credentials/{credential_id}/token"

# Test channel access
curl -H "Authorization: Bearer {access_token}" \
  "https://www.googleapis.com/youtube/v3/channels?part=snippet&mine=true"

# Test upload init
curl -X POST \
  -H "Authorization: Bearer {access_token}" \
  -H "Content-Type: application/json" \
  -d '{"snippet":{"title":"Test","description":"Test"},"status":{"privacyStatus":"private"}}' \
  "https://www.googleapis.com/upload/youtube/v3/videos?uploadType=resumable&part=snippet,status"
```

If the channel request succeeds (200) but upload returns 403:
- The account may have upload restrictions (brand accounts, age-restricted accounts)
- Try with a different Google account
- Check [YouTube API quota](https://console.cloud.google.com/apis/api/youtube.googleapis.com/quotas) — uploads cost 1600 units

**5. Check for parameter leakage in logs:**
Look at the transfer worker logs for the actual URL being called:
```bash
docker logs shs-w-transfer --tail 50
```
The upload URL should be clean — no extra query parameters beyond what YouTube's resumable upload session provides.

### OAuth Flow Not Working

**Re-authorize button missing:**
- Verify `GOOGLE_CLIENT_ID` and `GOOGLE_CLIENT_SECRET` are set as environment variables on the **API container**
- Restart the API after adding env vars
- Check that `/api/v1/oauth/providers` returns `"google": { "platform_configured": true }`

**Redirect URI mismatch:**
- The redirect URI in Google Cloud Console must exactly match: `{API_BASE_URL}/api/v1/oauth/google/callback`
- Common mistake: using the frontend URL instead of the API URL
- For local dev: `http://localhost:8000/api/v1/oauth/google/callback`

**Account picker not showing:**
- The OAuth flow includes `prompt=consent select_account` to force the account picker
- If it auto-selects an account, clear Google cookies or use an incognito window

### Upload Workflow Not Progressing

**Init Upload succeeds but Upload Bytes fails:**
- The Upload Bytes step must use `{{ step_name.response_headers.location }}` as its endpoint URL
- This URL contains the `upload_id` from Google — don't add any query parameters
- Check that the service's `parameterMapping` is `{}` (empty = zero query params)

**Step shows RUNNING but no progress:**
- Check the transfer worker logs: `docker logs shs-w-transfer --tail 50`
- The transfer worker handles file uploads on the `transfer_jobs` queue
- Verify the transfer worker container is running

**Step doesn't show as RUNNING:**
- PROCESSING status notifications require the API changes from commit `f3a097c6`
- Restart the API if recently updated

---

## Example: Upload Workflow

A typical YouTube upload workflow in Studio:

```
Step 1: Generate Image (leonardo.text_to_image or shs-comfyui)
  → Outputs: downloaded_files[0].virtual_path

Step 2: Create Video (shs_create_video)
  → Uses image from Step 1 as scene
  → Outputs: storage_url (video file in workspace)

Step 3: Init YouTube Upload (youtube_upload_video)
  → Parameters:
    - title: "My Video"
    - description: "Generated by Studio"
    - privacyStatus: "private"
  → Outputs: response_headers.location (upload URL)

Step 4: Upload Video Bytes (youtube_upload_video_bytes)
  → Parameters:
    - upload_url: {{ step3.response_headers.location }}
    - body_source_url: {{ step2.storage_url }}
  → Outputs: YouTube video resource (id, snippet, status)
```

### Key Configuration Details

**Init Upload (Step 3):**
- Endpoint: `https://www.googleapis.com/upload/youtube/v3/videos`
- Method: POST
- `parameterMapping.query`: `["uploadType", "part"]` — these go as query params
- Request body: `{ "snippet": { "title": "...", "description": "..." }, "status": { "privacyStatus": "..." } }`
- The response `Location` header contains the resumable upload URL

**Upload Bytes (Step 4):**
- Endpoint: `{{ parameters.upload_url }}` — dynamic from previous step
- Method: PUT
- `parameterMapping`: `{}` — empty means zero query params (upload URL is self-contained)
- Queue: `transfer_jobs` — handled by the transfer worker, not the general worker
- The file is streamed directly from workspace storage

---

## Quota

YouTube Data API v3 has a daily quota of **10,000 units** by default.

| Operation | Cost |
|-----------|------|
| Read (list, get, search) | 1 unit |
| Upload video | 1,600 units |
| Update/delete | 50 units |
| Set thumbnail | 50 units |

With the default quota, you can upload about **6 videos per day**. Request a [quota increase](https://support.google.com/youtube/contact/yt_api_form) for production use.

---

## Security Notes

- OAuth credentials are stored encrypted in the organization's credential store
- Platform OAuth credentials (env vars) are never exposed to users
- Each organization has completely isolated credentials
- Access tokens are refreshed automatically — refresh tokens are stored securely
- The platform uses the `offline` access type to ensure refresh tokens are always available
