**📡 RSS to Slack Automation with n8n
**
Automatically post new RSS feed articles to your Slack channel using n8n workflow automation

**🎯 Overview
**This project demonstrates how to build a fully automated RSS-to-Slack workflow using n8n. New articles from your favorite RSS feeds are automatically formatted and posted to a designated Slack channel every 15 minutes.
**✨ Features
**
⏰ Automatic polling - Checks RSS feeds every 15 minutes
🔄 Duplicate prevention - Only posts new articles
🎨 Beautiful formatting - Clean, professional Slack messages with Block Kit
🚀 Zero maintenance - Set it and forget it
📱 Real-time updates - Get notified as soon as new content is published


**🏗️ Architecture
**
┌─────────────┐    ┌──────────┐     ┌────────────┐     ┌──────┐    ┌──────────┐
│  Schedule   │───▶│   RSS    │───▶│   Remove   │───▶│ Code │───▶│   HTTP   │
│   Trigger   │    │   Read   │     │ Duplicates │     │ Node │    │ Request  │
│ (15 min)    │    │          │     │            │     │      │    │ (Slack)  │
└─────────────┘    └──────────┘     └────────────┘     └──────┘    └──────────┘
**Workflow Components
**
Schedule Trigger - Runs workflow every 15 minutes
RSS Feed Read - Fetches articles from RSS feed
Remove Duplicates - Filters out already-seen articles
Code (JavaScript) - Formats data for Slack Block Kit
HTTP Request - Sends formatted message to Slack webhook


**🚀 Quick Start
**
**Prerequisites
**
n8n instance (cloud or self-hosted)
Slack workspace with admin access
RSS feed URL

Step 1: Get Slack Webhook URL

Go to https://api.slack.com/apps
Click "Create New App" → "From scratch"
Name your app (e.g., "RSS Bot") and select your workspace
In the sidebar, click "Incoming Webhooks"
Toggle "Activate Incoming Webhooks" to ON
Click "Add New Webhook to Workspace"
Select the channel where posts should appear
Copy the webhook URL (starts with https://hooks.slack.com/services/...)

Step 2: Create n8n Workflow

Open n8n and create a new workflow
Add the following nodes in order:

Node 1: Schedule Trigger
Trigger Interval: Every 15 minutes
Node 2: RSS Feed Read
URL: https://techcrunch.com/feed/
(Or use any RSS feed URL you prefer)
Node 3: Remove Duplicates
Value to Dedupe On: link
Node 4: Code (JavaScript)
javascriptconst item = $input.first().json;

return [{
  json: {
    blocks: [
      {
        type: "section",
        text: {
          type: "mrkdwn",
          text: `*<${item.link}|${item.title}>*\n${(item.contentSnippet || "").slice(0, 200)}...`
        }
      },
      {
        type: "context",
        elements: [
          {
            type: "mrkdwn",
            text: `📅 ${item.pubDate}`
          }
        ]
      }
    ]
  }
}];
Node 5: HTTP Request
Method:              POST
URL:                 [Your Slack Webhook URL]
Send Body:           ON
Body Content Type:   Raw/Custom
Content Type:        application/json
Body (Expression):   {{ JSON.stringify($json) }}
Step 3: Test & Activate

Click "Test Workflow" to run it once
Check your Slack channel for the test message
Toggle the workflow to Active
Click Save

🎉 Done! Your bot is now live and will post new articles every 15 minutes.

**📚 RSS Feed Examples
**Here are some popular RSS feeds you can use:
SourceURLTechCrunchhttps://techcrunch.com/feed/Hacker Newshttps://hnrss.org/frontpageDev.tohttps://dev.to/feedWiredhttps://www.wired.com/feed/rssThe Vergehttps://www.theverge.com/rss/index.xmlArs Technicahttps://feeds.arstechnica.com/arstechnica/indexMIT Technology Reviewhttps://www.technologyreview.com/feed/

**🎨 Customization
**Change Update Frequency
In the Schedule Trigger node, modify the interval:

Every 5 minutes (for fast-moving feeds)
Every 30 minutes (for slower feeds)
Every 1 hour (for daily digests)

**Customize Slack Message Format
**Edit the Code node to change how messages appear:
javascript// Add author name
{
  type: "context",
  elements: [
    {
      type: "mrkdwn",
      text: `👤 ${item.creator} | 📅 ${item.pubDate}`
    }
  ]
}

// Add categories/tags
{
  type: "context",
  elements: [
    {
      type: "mrkdwn",
      text: `🏷️ ${item.categories?.join(', ')}`
    }
  ]
}

// Add emoji based on keywords
const emoji = item.title.toLowerCase().includes('ai') ? '🤖' : '📰';
text: `${emoji} *<${item.link}|${item.title}>*`
Multiple RSS Feeds
To monitor multiple feeds:

Duplicate the RSS Read node for each feed
Use a Merge node to combine them
Connect the merge output to Remove Duplicates


🔧 Troubleshooting
Error: "invalid_payload"
Problem: Slack webhook returns 400 error
Solution: Make sure HTTP Request body uses {{ JSON.stringify($json) }} in Expression mode
Error: "No items returned"
Problem: RSS Read node shows 0 items
Solution: Test the RSS URL in your browser - it should display XML content
Duplicate Messages
Problem: Same articles post multiple times
Solution: Verify Remove Duplicates node has "link" in "Value to Dedupe On" field
Messages Don't Appear in Slack
Problem: Workflow runs but nothing posts
Solution:

Check webhook URL is correct
Verify the Slack app has permission to post in the channel
Test the webhook directly with curl:

bashcurl -X POST -H 'Content-type: application/json' \
--data '{"text":"Test message"}' \
YOUR_WEBHOOK_URL

📊 Workflow Stats

Execution time: ~2-5 seconds per run
API calls: 1 per RSS feed + 1 per new article
Memory usage: Minimal (<10MB)
Scalability: Can handle 100+ feeds with proper scheduling


🤝 Contributing
Contributions are welcome! Here are some ideas:

 Add support for Discord webhooks
 Implement AI-powered content filtering
 Create a web UI for feed management
 Add sentiment analysis to articles
 Support for image extraction and posting
 Email digest option

How to Contribute

Fork this repository
Create a feature branch (git checkout -b feature/amazing-feature)
Commit your changes (git commit -m 'Add amazing feature')
Push to the branch (git push origin feature/amazing-feature)
Open a Pull Request


📝 License
This project is licensed under the MIT License - see the LICENSE file for details.

🙏 Acknowledgments

n8n - Workflow automation platform
Slack API - Slack integration
RSS feed providers - Content sources
