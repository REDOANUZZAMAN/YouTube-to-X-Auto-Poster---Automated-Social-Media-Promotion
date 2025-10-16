# 🎥 YouTube to X Auto-Poster - Automated Social Media Promotion

A lightweight n8n workflow that automatically detects new YouTube videos from your channel and posts engaging AI-generated tweets to X (Twitter) with video links. Perfect for content creators who want to grow their audience across platforms without manual posting.

![Status](https://img.shields.io/badge/status-active-success.svg)
![n8n](https://img.shields.io/badge/n8n-workflow-EA4B71?logo=n8n)
![YouTube](https://img.shields.io/badge/YouTube-FF0000?logo=youtube&logoColor=white)
![X](https://img.shields.io/badge/X-000000?logo=x&logoColor=white)

## ✨ Features

- **Automatic Detection** - Checks for new videos every 30 minutes
- **AI-Generated Posts** - ChatGPT creates engaging tweets (<140 chars)
- **YouTube Integration** - Fetches latest video metadata
- **X/Twitter Posting** - Auto-publishes with video link
- **Simple Setup** - Only 5 nodes, ready in minutes
- **Smart Timing** - 30-minute window prevents duplicate posts
- **Channel-Specific** - Monitors your YouTube channel only
- **Zero Manual Work** - Completely hands-off after setup

## 🔄 Workflow Overview

### Data Flow

```
Schedule Trigger
  (Every 30 minutes)
     ↓
Fetch Latest YouTube Videos
  (Published in last 30 min)
     ↓
AI Post Generation
  (ChatGPT 3.5)
     ↓
Post to X/Twitter
  (With video link)
```

## 🚀 Quick Start

### Prerequisites

- n8n instance (self-hosted or cloud)
- YouTube channel
- X/Twitter account
- OpenAI API key (GPT-3.5)

### YouTube API Setup

```
1. Go to Google Cloud Console
2. Create new project (or use existing)
3. Enable YouTube Data API v3
4. Create OAuth credentials
5. Add authorized redirect URI (n8n webhook)
6. Note Client ID and Secret
```

### X/Twitter API Setup

```
1. Visit developer.twitter.com
2. Create App (if not exists)
3. Enable OAuth 1.0a
4. Get API Key, API Secret, Access Token, Access Token Secret
5. Verify write permissions enabled
```

### Workflow Installation

1. **Import Workflow**
   ```bash
   1. Download "Post New YouTube Videos to X.json"
   2. In n8n: Import from File
   3. Upload JSON file
   ```

2. **Configure YouTube Channel**
   ```javascript
   // Fetch Latest Videos node
   channelId: "YOUR_YOUTUBE_CHANNEL_ID"
   
   // Get your channel ID:
   // 1. Go to youtube.com/account_advanced
   // 2. Copy "YouTube Channel ID"
   ```

3. **Add Credentials**
   - YouTube OAuth (Fetch Latest Videos)
   - OpenAI API (ChatGPT node)
   - X/Twitter API (Post to X)

4. **Test Workflow**
   - Click "Test workflow"
   - Check if latest video detected
   - Verify post generated
   - Confirm posted to X

5. **Activate**
   - Toggle workflow to "Active"
   - Will run every 30 minutes automatically

## ⚙️ Configuration

### Polling Frequency

```javascript
// Schedule Trigger node
interval: [
  { field: "minutes", minutesInterval: 30 }
]

// Options:
minutesInterval: 15  // Every 15 minutes
minutesInterval: 60  // Every hour
// Or use custom cron expression
```

### Time Window

```javascript
// Fetch Latest Videos node
publishedAfter: "={{ new Date(new Date().getTime() - 30 * 60000).toISOString() }}"

// Explanation:
30 * 60000 = 30 minutes in milliseconds

// Adjust window:
60 * 60000  // 1 hour
15 * 60000  // 15 minutes

// Must match or exceed polling frequency!
```

### Post Character Limit

```javascript
// Generate Post for X node
prompt: "Write an engaging post... of no more than 140 characters"

// Adjust limit:
"no more than 140 characters"  // Safe for X
"no more than 280 characters"  // Max X length
"no more than 100 characters"  // Short & punchy
```

### AI Model

```javascript
// Currently: GPT-3.5-Turbo (fast, cheap)
modelId: "gpt-3.5-turbo"

// Upgrade options:
modelId: "gpt-4o-mini"      // Better quality
modelId: "gpt-4"            // Best quality (expensive)
```

### Post Style

```javascript
// Customize prompt in Generate Post node:

// Current:
"Write an engaging post about my latest YouTube video"

// Casual:
"Write a fun, casual tweet about this new video"

// Professional:
"Write a professional announcement about this video release"

// With emojis:
"Write an engaging tweet with 2-3 relevant emojis"

// With hashtags:
"Write a tweet with 2 trending hashtags related to the content"
```

## 📁 Workflow Structure

```
YouTube to X Auto-Poster/
├── Trigger
│   └── Check Every 30 Min (Schedule Trigger)
│       - Cron: Every 30 minutes
│       - Starts workflow automatically
│
├── Fetch Data
│   └── Fetch Latest Videos (YouTube)
│       - Limit: 1 video
│       - Filter: channelId
│       - Time window: Last 30 minutes
│       - Returns: id, snippet (title, description)
│
├── AI Generation
│   └── Generate Post for X with ChatGPT (OpenAI)
│       - Model: GPT-3.5-Turbo
│       - Input: Title, description, video ID
│       - Output: Engaging tweet <140 chars
│       - Includes: youtu.be link
│
└── Publishing
    └── Post to X (Twitter)
        - Posts: AI-generated content
        - Includes: Video link
        - Immediate publication
```

**Total Nodes**: 5

## 🎯 How It Works

### Step 1: Scheduled Check (Every 30 minutes)

**Schedule Trigger**:
- Runs automatically
- No manual intervention
- Consistent timing
- Executes workflow

### Step 2: Video Detection

**YouTube API Call**:
```javascript
Query parameters:
- channelId: "YOUR_CHANNEL_ID"
- publishedAfter: "30 minutes ago"
- limit: 1
- order: date (newest first)

Returns if new video found:
{
  id: { videoId: "abc123xyz" },
  snippet: {
    title: "My Awesome Video Title",
    description: "Video description...",
    publishedAt: "2025-01-15T10:30:00Z"
  }
}

Returns empty if no new videos
```

**No New Video**: Workflow ends, waits for next trigger

**New Video Found**: Continues to AI generation

### Step 3: AI Post Generation

**ChatGPT Processing**:
```javascript
Input prompt:
"Write an engaging post about my latest YouTube video 
for X (Twitter) of no more than 140 characters in length. 
Link to the video at https://youtu.be/abc123xyz 
use this title and description: 
My Awesome Video Title 
Video description..."

AI generates example:
"🎥 Just dropped: My Awesome Video Title! 
Check it out → https://youtu.be/abc123xyz"

Character count: 137 ✅
```

**Output**: `message.content` field with generated text

### Step 4: X/Twitter Publishing

**X API Call**:
```javascript
POST /statuses/update
Body: {
  status: "🎥 Just dropped: My Awesome Video Title! 
          Check it out → https://youtu.be/abc123xyz"
}

Response: Tweet created
- Tweet ID
- Published timestamp
- Engagement metrics (0 initially)
```

**Result**: Tweet live on your X profile

## 🎨 Customization Examples

### Multiple Channels

```javascript
// Duplicate "Fetch Latest Videos" node
// Change channelId for each
// Merge outputs before AI generation

// Or loop through channel array:
const channels = [
  "CHANNEL_ID_1",
  "CHANNEL_ID_2",
  "CHANNEL_ID_3"
];
```

### Add Hashtags Dynamically

```javascript
// Modify AI prompt:
"Write an engaging post with 2 relevant hashtags 
based on video content"

// Or extract from video tags:
hashtags = video.snippet.tags.slice(0, 2)
  .map(tag => `#${tag}`)
  .join(' ');
```

### Cross-Post to Other Platforms

```javascript
// After X posting, add nodes:
- LinkedIn post
- Facebook post
- Instagram (via Buffer/Hootsuite)
- Discord webhook
- Telegram channel
```

### Video Thumbnail

```javascript
// Add image to X post:
1. Get thumbnail URL from YouTube
2. Download image (HTTP Request)
3. Post to X with media

// X API supports image uploads
media_ids: [uploaded_thumbnail_id]
```

### Analytics Tracking

```javascript
// Add UTM parameters to link:
video_url = `https://youtu.be/${videoId}?utm_source=twitter&utm_medium=social&utm_campaign=auto_post`

// Track in Google Analytics
```

### Smart Scheduling

```javascript
// Only post during optimal hours
const hour = new Date().getHours();
if (hour < 9 || hour > 17) {
  // Skip or delay until morning
}

// Best times: 9 AM - 5 PM weekdays
```

## 🛠️ Node Breakdown

| Node | Type | Purpose |
|------|------|---------|
| Check Every 30 Min | Schedule Trigger | Initiates workflow every 30 minutes |
| Fetch Latest Videos | YouTube | Gets newest video from channel |
| Generate Post for X | OpenAI | Creates AI-powered tweet text |
| Post to X | Twitter | Publishes tweet to X/Twitter |
| Sticky Note | Documentation | GitHub link |

**Total Active Nodes**: 4

## 🔧 Troubleshooting

### No Videos Detected

```javascript
// Check:
1. YouTube channel ID is correct
2. Time window matches polling (30 min)
3. Video was published recently
4. YouTube API credentials valid

// Test manually:
- Set publishedAfter to yesterday
- Should return recent videos
```

### AI Generation Fails

```javascript
// OpenAI node
1. Verify API key is valid
2. Check GPT-3.5 access
3. Test prompt manually
4. Check token limits

// Common issues:
- No credits in OpenAI account
- API key expired
- Model name incorrect
```

### X Post Fails

```javascript
// Twitter node
1. Check API credentials (4 keys needed)
2. Verify write permissions
3. Check character limit (<280)
4. Avoid duplicate posts

// Common errors:
- "403 Forbidden" → No write access
- "187 Status duplicate" → Same text posted
- "420 Rate limit" → Too many posts
```

### Duplicate Posts

```javascript
// Causes:
1. Time window too large (>30 min)
2. Polling frequency too high
3. Manual workflow runs

// Fix:
- Match window to polling frequency
- Add duplicate detection
- Check video ID against database
```

### Post Too Long

```javascript
// X character limits:
- Standard: 280 characters
- With link: Link counts as 23 chars
- Our limit: 140 chars (safe)

// If posts exceed:
1. Reduce character limit in prompt
2. Check AI is following instructions
3. Add post-processing truncation
```

### YouTube API Quota

```javascript
// YouTube API limits:
- 10,000 units/day (free)
- search.list costs: 100 units

// Our usage:
48 checks/day × 100 = 4,800 units (safe)

// If exceeded:
- Reduce polling frequency
- Request quota increase
- Use webhook instead (YouTube doesn't offer)
```

## 💡 Enhancement Ideas

### Immediate Improvements
- [ ] Add video thumbnail to tweet
- [ ] Include view count milestone
- [ ] Add emoji based on video category
- [ ] Thread for longer descriptions
- [ ] Pin important videos
- [ ] Tag relevant accounts

### Advanced Features
- [ ] A/B test different post styles
- [ ] Engagement analytics tracking
- [ ] Best time to post detection
- [ ] Automatic reply to comments
- [ ] Cross-platform posting (LinkedIn, FB)
- [ ] Video shorts to X videos
- [ ] Series detection (Part 1/2/3)

### AI Enhancements
- [ ] Generate multiple post variations
- [ ] Sentiment analysis for tone
- [ ] Trending hashtag suggestions
- [ ] Audience analysis for targeting
- [ ] Content category detection
- [ ] Hook generation from transcript
- [ ] Summary from video captions

### Integration Options
- [ ] Google Analytics tracking
- [ ] Buffer/Hootsuite scheduling
- [ ] Discord community notification
- [ ] Email digest to subscribers
- [ ] Notion content calendar
- [ ] Slack team notification

## 📊 Performance Metrics

### Execution Time (Per Check)
- **YouTube API call**: 1-2 seconds
- **AI generation**: 3-5 seconds
- **X posting**: 1-2 seconds
- **Total**: 5-10 seconds

**Note**: Most checks find no new video (instant)

### API Costs (Approximate)
- **YouTube API**: Free (within quota)
- **OpenAI GPT-3.5**: $0.001-0.002 per post
- **X API**: Free

**Monthly cost** (assuming 10 videos): ~$0.02

### Frequency Options
- **Every 15 min**: 96 checks/day, instant posting
- **Every 30 min**: 48 checks/day, <30 min delay (recommended)
- **Every hour**: 24 checks/day, <60 min delay

## 🔒 Security Best Practices

- **API Keys**: Store in n8n credentials only
- **OAuth Tokens**: Use n8n's OAuth flow
- **X Permissions**: Request write-only (not DM access)
- **YouTube Scope**: Request minimum (readonly)
- **Rate Limiting**: Respect API quotas
- **Content Review**: Test AI output before going live
- **Account Security**: Enable 2FA on all accounts

## 📝 Credentials Setup

### YouTube OAuth

```
1. Google Cloud Console → APIs & Services
2. Create OAuth 2.0 Client ID
3. Authorized redirect URIs: 
   https://your-n8n.com/rest/oauth2-credential/callback
4. Add to n8n YouTube credentials
5. Authorize access
```

### X/Twitter API

```
1. developer.twitter.com → Developer Portal
2. Create App (or use existing)
3. Get keys from "Keys and tokens":
   - API Key
   - API Secret Key
   - Access Token
   - Access Token Secret
4. Add all 4 to n8n Twitter credentials
```

### OpenAI API

```
1. platform.openai.com → API Keys
2. Create new secret key
3. Copy immediately (won't show again)
4. Add to n8n OpenAI credentials
5. Monitor usage in dashboard
```

## 📖 Additional Resources

- [YouTube Data API Docs](https://developers.google.com/youtube/v3)
- [X/Twitter API Reference](https://developer.twitter.com/en/docs/twitter-api)
- [OpenAI API Documentation](https://platform.openai.com/docs)
- [n8n YouTube Integration](https://docs.n8n.io/integrations/builtin/app-nodes/n8n-nodes-base.youtube/)
- [n8n Twitter Integration](https://docs.n8n.io/integrations/builtin/app-nodes/n8n-nodes-base.twitter/)

## 📄 License

This workflow is open source and available under the [MIT License](LICENSE).

## 🤝 Contributing

Contributions welcome! Focus areas:

- Multi-platform posting
- Better post templates
- Engagement tracking
- Video thumbnail support
- Analytics integration

1. Fork repository
2. Create feature branch (`git checkout -b feature/InstagramPosts`)
3. Commit changes (`git commit -m 'Add Instagram cross-post'`)
4. Push to branch (`git push origin feature/InstagramPosts`)
5. Open Pull Request

## 👨‍💻 Author

**Redoanuzzaman**
- GitHub: [@REDOANUZZAMAN](https://github.com/REDOANUZZAMAN)
- Email: redoanuzzaman707@gmail.com
- Website: [redoan.dev](https://redoan.dev)

## 🙏 Acknowledgments

- n8n community for automation platform
- OpenAI for GPT-3.5
- YouTube and X/Twitter for APIs
- Content creators using this workflow

## 💖 Show Your Support

Give a ⭐️ if this workflow helps grow your audience!

## 🎯 Use Cases

- **YouTubers**: Auto-promote every upload
- **Course Creators**: Share educational content
- **Vloggers**: Build Twitter following
- **Tutorial Channels**: Drive traffic to videos
- **Gaming Streamers**: Announce new content
- **Podcasters**: Promote video episodes
- **Businesses**: Content marketing automation

---

Made with 🎥 and social media automation

**Last Updated:** October 2025
