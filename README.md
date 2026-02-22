# RSS to Slack Automation with n8n

<div align="center">
  
Automatically post new RSS feed articles to your Slack channel using n8n workflow automation

<img width="1598" height="395" alt="image" src="https://github.com/user-attachments/assets/76d985a0-1187-4f91-94dd-c04095893eec" />

</div>

---


## 🎯 Overview

This project demonstrates how to build a fully automated **RSS-to-Slack** workflow using n8n. New articles from your favorite RSS feeds are automatically formatted and posted to a designated Slack channel every 15 minutes.

---

## ✨ Features

| Feature | Description |
|---------|-------------|
| ⏰ **Automatic polling** | Checks RSS feeds every 15 minutes |
| 🔄 **Duplicate prevention** | Only posts new articles |
| 🎨 **Beautiful formatting** | Clean, professional Slack messages with Block Kit |
| 🚀 **Zero maintenance** | Set it and forget it |
| 📱 **Real-time updates** | Get notified as soon as new content is published |

---

## 🏗️ Architecture

```
┌─────────────┐    ┌──────────┐    ┌────────────┐    ┌──────┐    ┌──────────┐
│  Schedule   │───▶│   RSS    │───▶│   Remove   │───▶│ Code │───▶│   HTTP   │
│   Trigger   │    │   Read   │    │ Duplicates │    │ Node │    │ Request  │
│ (15 min)    │    │          │    │            │    │      │    │ (Slack)  │
└─────────────┘    └──────────┘    └────────────┘    └──────┘    └──────────┘
```

### Workflow Components

1. **Schedule Trigger** - Runs workflow every 15 minutes
2. **RSS Feed Read** - Fetches articles from RSS feed
3. **Remove Duplicates** - Filters out already-seen articles
4. **Code (JavaScript)** - Formats data for Slack Block Kit
5. **HTTP Request** - Sends formatted message to Slack webhook

---

## 🚀 Quick Start

### Prerequisites

- ✅ n8n instance (cloud or self-hosted)
- ✅ Slack workspace with admin access
- ✅ RSS feed URL

---

### Step 1: Get Slack Webhook URL

1. Go to [https://api.slack.com/apps](https://api.slack.com/apps)
2. Click **"Create New App"** → **"From scratch"**
3. Name your app (e.g., "RSS Bot") and select your workspace
4. In the sidebar, click **"Incoming Webhooks"**
5. Toggle **"Activate Incoming Webhooks"** to ON
6. Click **"Add New Webhook to Workspace"**
7. Select the channel where posts should appear
8. **Copy the webhook URL** (starts with `https://hooks.slack.com/services/...`)

---

### Step 2: Create n8n Workflow

Open n8n and create a new workflow. Add the following nodes in order:

#### 📌 Node 1: Schedule Trigger
```
Trigger Interval: Every 15 minutes
```

#### 📌 Node 2: RSS Feed Read
```
URL: https://techcrunch.com/feed/
```
*(Or use any RSS feed URL you prefer)*

#### 📌 Node 3: Remove Duplicates
```
Value to Dedupe On: link
```

#### 📌 Node 4: Code (JavaScript)
```javascript
// Process ALL items, not just the first one
const items = $input.all();

// Map each RSS item to Slack format
return items.map(item => ({
  json: {
    blocks: [
      {
        type: "section",
        text: {
          type: "mrkdwn",
          text: `*<${item.json.link}|${item.json.title}>*\n${(item.json.contentSnippet || "").slice(0, 200)}...`
        }
      },
      {
        type: "context",
        elements: [
          {
            type: "mrkdwn",
            text: `📅 ${item.json.pubDate}`
          }
        ]
      }
    ]
  }
}));
```

#### 📌 Node 5: HTTP Request
| Setting | Value |
|---------|-------|
| **Method** | `POST` |
| **URL** | `[Your Slack Webhook URL]` |
| **Send Body** | ON |
| **Body Content Type** | Raw/Custom |
| **Content Type** | `application/json` |
| **Body (Expression)** | `{{ JSON.stringify($json) }}` |

---

### Step 3: Test & Activate

1. Click **"Test Workflow"** to run it once
2. Check your Slack channel for the test message
3. Toggle the workflow to **Active**
4. Click **Save**

---

## 🎉 Done! Your bot is now live and will post new articles every 15 minutes.

---

## 📚 RSS Feed Examples

Here are some popular RSS feeds you can use:

| Source | URL |
|--------|-----|
| **TechCrunch** | `https://techcrunch.com/feed/` |
| **Hacker News** | `https://hnrss.org/frontpage` |
| **Dev.to** | `https://dev.to/feed` |
| **Wired** | `https://www.wired.com/feed/rss` |
| **The Verge** | `https://www.theverge.com/rss/index.xml` |
| **Ars Technica** | `https://feeds.arstechnica.com/arstechnica/index` |
| **MIT Technology Review** | `https://www.technologyreview.com/feed/` |

---

## 🎨 Customization

### Change Update Frequency

In the Schedule Trigger node, modify the interval:

- ⚡ **Every 5 minutes** - for fast-moving feeds
- ⏱️ **Every 30 minutes** - for slower feeds
- 📆 **Every 1 hour** - for daily digests

### Customize Slack Message Format

Edit the Code node to change how messages appear:

**Add author name:**
```javascript
{
  type: "context",
  elements: [
    {
      type: "mrkdwn",
      text: `👤 ${item.creator} | 📅 ${item.pubDate}`
    }
  ]
}
```

**Add categories/tags:**
```javascript
{
  type: "context",
  elements: [
    {
      type: "mrkdwn",
      text: `🏷️ ${item.categories?.join(', ')}`
    }
  ]
}
```

**Add emoji based on keywords:**
```javascript
const emoji = item.title.toLowerCase().includes('ai') ? '🤖' : '📰';
text: `${emoji} *<${item.link}|${item.title}>*`
```

### Multiple RSS Feeds

To monitor multiple feeds:

1. Duplicate the RSS Read node for each feed
2. Use a **Merge** node to combine them
3. Connect the merge output to Remove Duplicates

---

## 🔧 Troubleshooting

| Error | Problem | Solution |
|-------|---------|----------|
| **"invalid_payload"** | Slack webhook returns 400 error | Make sure HTTP Request body uses `{{ JSON.stringify($json) }}` in Expression mode |
| **"No items returned"** | RSS Read node shows 0 items | Test the RSS URL in your browser - it should display XML content |
| **Duplicate Messages** | Same articles post multiple times | Verify Remove Duplicates node has "link" in "Value to Dedupe On" field |
| **Messages Don't Appear** | Workflow runs but nothing posts | Check webhook URL, verify Slack app permissions, test webhook directly: |

**Test webhook directly with curl:**
```bash
curl -X POST -H 'Content-type: application/json' \
--data '{"text":"Test message"}' \
YOUR_WEBHOOK_URL
```

---

## 📊 Workflow Stats

- **Execution time:** ~2-5 seconds per run
- **API calls:** 1 per RSS feed + 1 per new article
- **Memory usage:** Minimal (<10MB)
- **Scalability:** Can handle 100+ feeds with proper scheduling

---

## 🤝 Contributing

Contributions are welcome! Here are some ideas:

- ✨ Add support for Discord webhooks
- 🧠 Implement AI-powered content filtering
- 🌐 Create a web UI for feed management
- 📊 Add sentiment analysis to articles
- 🖼️ Support for image extraction and posting
- 📧 Email digest option

### How to Contribute

1. Fork this repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

---

## 📝 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

---

## 🙏 Acknowledgments

- [n8n](https://n8n.io/) - Workflow automation platform
- [Slack API](https://api.slack.com/) - Slack integration
- RSS feed providers - Content sources

---

## 📧 Contact

Have questions or suggestions? Feel free to:

- [Open an issue](https://github.com/AIMANTAHIR04/AIMAN-S-WORKFLOW/issues)
- Submit a pull request
- Reach out on Twitter

---

## ⭐ Show Your Support

If this project helped you, please give it a **⭐️ on GitHub**!

---

<div align="center">
  <b>Built with ❤️ using n8n</b>
</div>

---
