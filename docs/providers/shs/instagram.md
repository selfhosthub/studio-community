# Instagram Provider

Publish content, manage comments, and track insights on Instagram Business and Creator accounts.

## How It Works

The Instagram provider connects to the Instagram Graph API via Facebook OAuth. It supports:

- **Content Publishing** - Images, videos/Reels, and carousels
- **Media Management** - List, get details, check processing status
- **Engagement** - Comments, replies, hide/delete moderation
- **Analytics** - Post insights and account-level metrics
- **Discovery** - Hashtag search, business discovery, mentions

## Authentication

Instagram uses **Facebook OAuth** authentication. You need an Instagram Business or Creator account linked to a Facebook Page.

### Setup

1. Go to [developers.facebook.com](https://developers.facebook.com/)
2. Create a Facebook App (type: Business)
3. Add the **Instagram Graph API** product
4. Configure OAuth redirect URI to your Studio instance
5. Required permissions:
   - `instagram_basic` - Read profile and media
   - `instagram_content_publish` - Publish posts
   - `instagram_manage_comments` - Read/write comments
   - `instagram_manage_insights` - Access analytics
   - `pages_show_list` - List connected pages
   - `pages_read_engagement` - Read page engagement
6. In Studio, go to **Providers > Instagram > Credentials**
7. Click **Add Credential** and complete the OAuth flow

### Requirements

- Instagram **Business** or **Creator** account (not Personal)
- Facebook Page linked to the Instagram account
- Facebook App with Instagram Graph API enabled

## Available Services

### Content Publishing

| Service | Description |
|---------|-------------|
| **Create Image Container** | Prepare an image post for publishing |
| **Create Video/Reel Container** | Prepare a video or Reel (async processing) |
| **Create Carousel Container** | Combine child containers into a carousel |
| **Check Container Status** | Poll processing status for video containers |
| **Publish Media** | Publish a prepared container to your feed |

**Publishing flow:** Create Container > (Poll Status for video) > Publish Media

### Media & Account

| Service | Description |
|---------|-------------|
| **Get Account Info** | Retrieve connected account details |
| **List Media** | List posts with engagement metrics |
| **Get Media Details** | Get detailed info for a specific post |

### Analytics

| Service | Description |
|---------|-------------|
| **Get Post Insights** | Impressions, reach, likes, comments, shares, saves |
| **Get Account Insights** | Account-level metrics over a time period |

### Comments & Engagement

| Service | Description |
|---------|-------------|
| **List Post Comments** | Retrieve comments including nested replies |
| **Reply to Comment** | Post a reply to a comment |
| **Hide/Unhide Comment** | Moderate comments by hiding or unhiding |
| **Delete Comment** | Remove a comment |

### Discovery & Search

| Service | Description |
|---------|-------------|
| **Search Hashtag** | Find hashtag ID by name |
| **Hashtag Top Media** | Top-performing media for a hashtag |
| **Hashtag Recent Media** | Most recent media for a hashtag |
| **Get Mentioned Media** | Media where you are tagged |
| **Get Mentioned Comments** | Comments mentioning you |
| **Business Discovery** | Look up another business account's public profile |

---

## Example Workflows

### Publish an Image Post

**Use case:** Post an image to Instagram

```
Trigger: Manual or Schedule
  |
Step 1: Create Image Container
  - image_url: "https://example.com/photo.jpg"
  - caption: "Check out our latest product!"
  |
Step 2: Publish Media
  - creation_id: {{ step1.id }}
```

### Publish a Video/Reel

**Use case:** Post a Reel with async processing

```
Trigger: Manual
  |
Step 1: Create Video Container
  - video_url: "https://example.com/video.mp4"
  - caption: "New Reel!"
  - media_type: "REELS"
  |
Step 2: Poll Service (Core)
  - Check Container Status until FINISHED
  |
Step 3: Publish Media
  - creation_id: {{ step1.id }}
```

### Daily Engagement Report

**Use case:** Track post performance

```
Trigger: Schedule (daily at 9am)
  |
Step 1: List Media (last 10 posts)
  |
Step 2: Get Post Insights (for each)
  |
Step 3: Summarize with AI
  |
Step 4: Send report via Slack/Email
```

---

## Multi-Step Workflows

> **Plus catalog:** Plus workflows and video walkthroughs come with a [SelfHost Innovators membership](https://www.skool.com/selfhostinnovators).

### Auto-Reply to Comments

Automatically reply to comments with AI-generated responses:

1. **List Comments** - Get new comments on recent posts
2. **Filter** - Skip already-replied comments
3. **Generate Reply** - Use AI to draft contextual reply
4. **Reply to Comment** - Post the reply
5. **Log** - Track replies for review

### Hashtag Research Pipeline

Research trending content in your niche:

1. **Search Hashtag** - Get hashtag ID
2. **Get Top Media** - Fetch top-performing posts
3. **Analyze Content** - AI-analyze what performs well
4. **Generate Report** - Create content strategy recommendations

### Cross-Platform Content Publisher

Publish content to Instagram and other platforms simultaneously:

1. **Prepare Content** - Format for each platform
2. **Create Instagram Container** - Prepare image/video
3. **Publish to Instagram** - Post to feed
4. **Post to other platforms** - TikTok, YouTube, etc.
5. **Track Performance** - Monitor engagement across platforms

---

## Troubleshooting

| Error | Solution |
|-------|----------|
| "Invalid OAuth token" | Re-authorize via Studio credentials page |
| "Permission denied" | Check Facebook App has required permissions |
| "Media not ready" | Poll container status until FINISHED before publishing |
| "Invalid image URL" | Image must be publicly accessible HTTPS URL |
| "Carousel needs 2-10 children" | Create 2-10 child containers before carousel |

## Tips

1. **Video Processing** - Videos require async processing; always poll status before publishing
2. **Image URLs** - Must be publicly accessible HTTPS URLs (no localhost or private networks)
3. **Carousel Order** - Child container IDs are published in the order provided
4. **Rate Limits** - Instagram API has rate limits per user; space out bulk operations
5. **Insights Delay** - Post insights may take a few hours to populate after publishing
6. **Hashtag Limits** - You can search 30 unique hashtags per 7-day rolling window

## Terms

Your use of Instagram is governed by Instagram's own terms, not by Studio's: [https://developers.facebook.com/terms](https://developers.facebook.com/terms). Costs, rate limits, content-ownership rules, and acceptable-use policies are set by the provider. You are responsible for complying with them and for any charges you incur. See LEGAL.md in the Studio repository.
