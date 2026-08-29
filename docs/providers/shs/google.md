# Google Provider

Connect to Google Workspace apps including Sheets, Drive, Gmail, and Calendar.

## Authentication

Google uses **OAuth2** for authentication. Each organization provides their own Google OAuth credentials, giving you full control over your data and API usage.

### Setup Overview

1. Create a Google Cloud project with OAuth app
2. Add a credential in Studio with your client ID and secret
3. Click "Authorize" to complete the OAuth flow

### Step 1: Create Google Cloud Project

1. Go to [Google Cloud Console](https://console.cloud.google.com/)
2. Create a new project or select an existing one
3. Enable the APIs you need:
   - **Google Sheets API** - for spreadsheet operations
   - **Google Drive API** - for file operations
   - **Gmail API** - for email operations
   - **Google Calendar API** - for calendar operations

### Step 2: Configure OAuth Consent Screen

1. Go to **APIs & Services > OAuth consent screen**
2. Choose **External** (or Internal for Google Workspace)
3. Fill in required fields:
   - App name: Your app name
   - User support email: Your email
   - Developer contact: Your email
4. Add scopes:
   - `https://www.googleapis.com/auth/spreadsheets`
   - `https://www.googleapis.com/auth/drive`
   - `https://www.googleapis.com/auth/gmail.modify`
   - `https://www.googleapis.com/auth/calendar`
5. Add test users if in testing mode

### Step 3: Create OAuth Credentials

1. Go to **APIs & Services > Credentials**
2. Click **Create Credentials > OAuth client ID**
3. Application type: **Web application**
4. Add authorized redirect URI:
   ```
   https://your-studio-domain.com/api/v1/oauth/google/callback
   ```
   Replace `your-studio-domain.com` with your actual Studio API domain.
5. Copy the **Client ID** and **Client Secret**

### Step 4: Add Credential in Studio

1. Go to **Providers > Google > Credentials**
2. Click **Add Credential**
3. Enter your **OAuth Client ID** and **Client Secret** from Google Cloud Console
4. Save the credential
5. Click the **Authorize** button on the credential
6. Sign in with your Google account and grant permissions
7. You're connected! The credential will show "Connected" status.

## Available Services

### Google Sheets

| Service | Description |
|---------|-------------|
| **Get Rows** | Read rows from a specified range |
| **Append Rows** | Add rows after the last row with content |
| **Update Rows** | Overwrite data in a specified range |
| **Clear Rows** | Remove data but keep formatting |
| **Get Spreadsheet Info** | Get metadata about sheets and properties |

**Finding the Spreadsheet ID:**
```
https://docs.google.com/spreadsheets/d/[SPREADSHEET_ID]/edit
```

### Google Drive

| Service | Description |
|---------|-------------|
| **List Files** | Search and list files/folders |
| **Upload File** | Upload new files |
| **Download File** | Download file content |
| **Get File Info** | Get file metadata |

### Gmail

| Service | Description |
|---------|-------------|
| **Send Email** | Send emails with text or HTML content |
| **List Emails** | Search and list emails |
| **Get Email** | Get full email content by ID |

### Google Calendar

| Service | Description |
|---------|-------------|
| **List Events** | Get calendar events with time range and search filtering |
| **Get Event** | Get details of a specific event |
| **Create Event** | Create new calendar events with attendees and reminders |
| **Update Event** | Modify existing events |
| **Delete Event** | Remove events from calendar |
| **Quick Add Event** | Create events from natural language (e.g., "Meeting tomorrow at 3pm") |

**Common Use Cases:**
- **Meeting Scheduled → Slack Notification**: When a new calendar event is created, send details to Slack
- **Event Reminder → Email**: Send reminder emails 24 hours before events
- **Form Submission → Calendar Event**: Create calendar events from form responses
- **Availability Checker**: List events in a time range to check availability

## Token Management

- **Access tokens** expire after 1 hour
- **Refresh tokens** are used automatically to get new access tokens
- If refresh fails, you will need to re-authorize

To manually refresh tokens:
```
POST /api/v1/oauth/google/refresh/{credential_id}
```

## Troubleshooting

| Error | Solution |
|-------|----------|
| "Credential not found" | Ensure you've created a credential with client_id and client_secret first |
| "Missing client_id or client_secret" | Edit your credential and add both OAuth app credentials from Google Cloud Console |
| "Access denied" | User denied permissions or consent screen not configured correctly |
| "Invalid redirect URI" | Add the callback URL shown in the error to your Google Cloud Console OAuth app |
| "Token refresh failed" | Re-authorize by clicking the Authorize button (permissions may have been revoked) |
| "Needs Authorization" status | Click the Authorize button to complete the OAuth flow |

## Security Notes

- Your OAuth credentials (client_id, client_secret) are stored encrypted in your organization's credential store
- Access and refresh tokens are automatically managed and stored securely
- Each organization has completely isolated credentials
- The platform super admin has no access to your OAuth credentials or tokens

## Terms

Your use of Google is governed by Google's own terms, not by Studio's: [https://developers.google.com/terms](https://developers.google.com/terms). Costs, rate limits, content-ownership rules, and acceptable-use policies are set by the provider. You are responsible for complying with them and for any charges you incur. See LEGAL.md in the Studio repository.
