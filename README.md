# Maritime Supply Chain Monitor

**Automated maritime disruption intelligence system that monitors global ports, shipping routes, and vessel incidents in real-time.**

![n8n](https://img.shields.io/badge/n8n-Workflow-orange)
![Airtable](https://img.shields.io/badge/Airtable-Database-blue)
![OpenRouter](https://img.shields.io/badge/OpenRouter-AI-green)

---

## 🎯 Project Overview

This automated system aggregates maritime news from multiple specialized sources, classifies disruptions by severity using AI, and delivers daily intelligence reports via email. Built for logistics professionals, freight forwarders, and supply chain managers who need to stay ahead of port strikes, vessel incidents, and route blockages.

### Key Features

- **Multi-Source Aggregation**: RSS feeds from Maritime Executive, The Loadstar, GAC Hot Port News, and ICC piracy reports
- **AI-Powered Classification**: Batch-processes articles using OpenRouter LLM (RED/YELLOW/GREEN severity)
- **Smart Deduplication**: URL-based filtering prevents duplicate storage (95%+ efficiency)
- **Automated Daily Digest**: HTML email with color-coded disruption alerts
- **Persistent Storage**: Airtable database for historical tracking and analytics

### Sample Output

📊 **Daily Metrics** (typical run):
- 95 articles fetched → 2-30 new (duplicates filtered)
- Classification: ~25% RED, ~25% YELLOW, ~50% GREEN
- Email delivery: <1 minute total runtime

---

## 🏗️ Architecture

```
┌─────────────────┐
│  RSS Feeds      │ Maritime Executive, Loadstar, GAC, ICC
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  Normalize      │ Standardize all formats to unified JSON
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  Deduplicate    │ Check against Airtable history (URL matching)
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  AI Classify    │ Batch process (20/call) → RED/YELLOW/GREEN
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  Store          │ Airtable with timestamps, classifications
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  Email Digest   │ HTML report sent to subscribers
└─────────────────┘
```

---

## 🚀 Quick Start

### Prerequisites

- **n8n** (self-hosted or cloud): [Installation guide](https://docs.n8n.io/hosting/)
- **Airtable account** (free tier works): [Sign up](https://airtable.com/signup)
- **OpenRouter API key** (free tier: 10 requests/day): [Get key](https://openrouter.ai/)
- **NewsData.io API key** (optional, for broader coverage): [Get key](https://newsdata.io/)

### Installation

1. **Import the workflow into n8n**
   ```bash
   # Download the workflow file
   wget https://raw.githubusercontent.com/YOUR_USERNAME/maritime-supply-chain-monitor/main/maritime-supply-chain-monitor.json

   # In n8n: Settings → Import from File → Select maritime-supply-chain-monitor.json
   ```

2. **Set up Airtable**
   - Create a new base called "Supply Chain Monitor"
   - Create a table "Articles" with these fields:
     - `article_url` (Single line text, Primary)
     - `title` (Single line text)
     - `summary` (Long text)
     - `source` (Single line text)
     - `published_date` (Date)
     - `first_seen_date` (Date)
     - `classification` (Single select: RED, YELLOW, GREEN)
     - `sent_in_email` (Checkbox)
   - Create a table "Users" with field: `email` (Email)

3. **Configure credentials in n8n**
   - **Airtable**: Settings → Credentials → Add Airtable Personal Access Token
   - **OpenRouter**: Add HTTP Header Auth with `Authorization: Bearer YOUR_KEY`
   - **SMTP**: Add your email provider credentials

4. **Update workflow parameters**
   - `Newsdata.io API` node: Replace `YOUR_NEWSDATA_API_KEY` in URL
   - `Send email` node: Replace `your-email@example.com` with your sender address

5. **Test the workflow**
   ```
   Click "Execute Workflow" → Verify articles appear in Airtable → Check email inbox
   ```

6. **Schedule daily runs**
   - Edit "Schedule Trigger" node → Set time (e.g., 7:00 AM)
   - Activate workflow

---

## 📊 Data Sources

| Source | Type | Coverage | Update Frequency |
|--------|------|----------|------------------|
| **Maritime Executive** | RSS | General maritime news, vessel incidents, regulations | ~10-30/day |
| **The Loadstar** | RSS | Freight rates, port congestion, logistics analysis | ~5-15/day |
| **GAC Hot Port News** | Web scraping | Port-specific operational notices (berth closures, strikes) | ~3-10/day |
| **ICC Piracy Reports** | API | Security incidents (hijacking, armed robbery at sea) | ~2-5/week |
| **NewsData.io** | API (optional) | Broader maritime coverage, filtered by keywords | Variable |

---

## 🧠 AI Classification Logic

Articles are classified using a batched LLM prompt (20 articles per API call):

- **RED (Critical)**: Port closures, route blockages, strikes beginning, vessel attacks, major accidents
- **YELLOW (Monitor)**: Delays, congestion warnings, strike threats, developing weather events
- **GREEN (Positive/Neutral)**: Operations resumed, strike averted, new routes, corporate appointments, sustainability initiatives

**Model**: `deepseek-r1t2-chimera:free` (OpenRouter) with 2-second delay between batches to respect rate limits.

---

## 🔧 Customization

### Add New RSS Feed

1. Add "RSS Feed Read" node
2. Connect to "Merge All Sources"
3. Update "Normalize All Formats" code to handle the new format:

```javascript
// Add to CASE 4 in determineSource():
if (url.includes('newsource.com')) return 'New Source Name';
```

### Change Classification Criteria

Edit the prompt in "Build Classification Prompt" node:

```javascript
RULES:
- RED: [Your custom critical criteria]
- YELLOW: [Your custom warning criteria]
- GREEN: [Your custom positive/neutral criteria]
```

### Modify Email Design

Edit "Build Email HTML" node's `generateArticleItem()` function for custom styling.

---

## 📈 Performance & Costs

**Runtime**: ~2-5 minutes per daily execution  
**API Costs** (per month, ~30 days):
- OpenRouter (150-300 requests): $0 (free tier) or ~$2-5 (paid)
- NewsData.io (30 requests): $0 (free tier)
- Airtable (storage): $0 (free tier covers ~1,200 articles/month)
- SMTP: $0 (using Gmail/Outlook)

**Total**: $0-5/month depending on volume

---

## 🛠️ Troubleshooting

### "Rate limit exceeded" error
- **Cause**: OpenRouter free tier limit (10 requests/day)
- **Fix**: Add $10 credit to OpenRouter or reduce batch size to 30 articles

### Workflow returns 0 new articles
- **Normal**: RSS feeds return last 50-100 items; most are duplicates from previous runs
- **Expected**: 2-30 truly new articles per day (depending on news volume)

### Email not received
- Check "Get Active Users" Airtable query returns users
- Verify SMTP credentials
- Check spam folder

### Articles classified incorrectly
- Review "Build Classification Prompt" → adjust rules
- Test with sample: `console.log(prompt)` in code node

---

## 📁 Project Structure

```
maritime-supply-chain-monitor/
├── maritime-supply-chain-monitor.json   # Main n8n workflow
├── README.md                            # This file
├── LICENSE                              # MIT License
└── .github/
    └── images/                          # Screenshots for README
```

---

## 🎥 Demo

*Coming soon: Video walkthrough of setup and sample email digest*

---

## 🤝 Contributing

Improvements welcome! Areas for contribution:
- Additional data sources (AIS tracking, port APIs)
- Regional filters (Asia-Pacific, EU, Americas)
- Slack/Teams integration for alerts
- Tableau/Power BI dashboard templates

---

## 📝 License

MIT License - see [LICENSE](LICENSE) file for details

---

## 👤 Author

**Laurynas**
- Project built as portfolio demonstration for Upwork
- Specialized in n8n automation, AI workflows, supply chain tech
- Available for custom maritime/logistics automation projects

---

## 🔗 Related Projects

- [ICC Piracy Scraper](https://github.com/YOUR_USERNAME/piracy-scraper) - Data source for this project

---

## 📧 Contact

For custom implementations or consulting: [Your Contact Method]

---

**⭐ If this project helped you, consider starring it on GitHub!**
