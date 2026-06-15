# AI-Powered Deal Sourcing and CRM Automation — n8n + HubSpot

<img width="1239" height="581" alt="1" src="https://github.com/user-attachments/assets/0669f967-0293-4144-8c3e-739862c34554" /><img width="1133" height="568" alt="2" src="https://github.com/user-attachments/assets/09e2b239-0e8d-4918-9a6a-f7f697c0a0a0" /><img width="933" height="501" alt="3" src="https://github.com/user-attachments/assets/8a6964b4-c32a-4a59-af6a-03601780ea06" />




An end-to-end business acquisition deal sourcing system built for a private equity / independent sponsor firm. The system automatically monitors broker emails and business-for-sale marketplaces, scores each opportunity using AI, pushes qualified deals into HubSpot, and delivers daily and weekly pipeline reports via Telegram — with zero manual effort.

---

## What It Does

- Monitors Gmail inbox for broker emails and classifies them using AI
- Scrapes BizBuySell and BizQuest via Apify for new business listings
- Extracts key deal fields: company name, industry, location, revenue, EBITDA, asking price, broker info
- Scores each deal from 0 to 10 using a custom acquisition criteria model
- Checks for duplicates before pushing to HubSpot CRM
- Creates deals in HubSpot with all extracted fields populated
- Sends instant Telegram alerts for high-fit deals (score 7+)
- Delivers a daily digest every morning with the top 10 deals by fit score
- Delivers a weekly pipeline report with source breakdown and average score

---

## Tech Stack

- **n8n** — Workflow automation and orchestration
- **OpenRouter (DeepSeek)** — AI model for email classification and deal analysis
- **HubSpot CRM** — Deal storage, pipeline management, and reporting
- **Apify** — Web scraping for BizBuySell and BizQuest marketplaces
- **Gmail** — Email monitoring and deal intake
- **Telegram Bot** — Real-time alerts and scheduled reports
- **JavaScript (Code Node)** — Data parsing, date logic, report formatting

---

## Workflow Overview

### Workflow 1 — Email Deal Intake (`workflow_1_email_intake.json`)

```
Schedule Trigger (hourly)
  → Gmail (fetch unread emails)
  → Loop Over Items
  → Code Node (extract subject, sender, body)
  → AI Agent (classify: is this a business acquisition email?)
  → If YES → AI Chain (extract deal fields + score)
  → Code Node (parse JSON response)
  → If fit score > 7 → HubSpot duplicate check
      → If new deal → HubSpot Create Deal
          → Telegram Alert
      → If duplicate → skip
  → Mark email as read → continue loop
```

**AI Classification logic:** The AI checks if the email is a real M&A opportunity (broker outreach, business for sale, acquisition target) and filters out newsletters, cold sales, job offers, and general marketing.

**AI Scoring logic:**
- Revenue $500K+ → +2 points
- EBITDA margin 15%+ → +2 points
- Asking price less than 5x EBITDA → +2 points
- USA location → +1 point
- Low or no red flags → +2 points
- Growth potential visible → +1 point

---

### Workflow 2 — Marketplace Scraper (`workflow_2_marketplace_scraper.json`)

```
Schedule Trigger (daily 9AM)
  → BizBuySell (Apify scraper) + BizQuest (Apify scraper) [parallel]
  → Loop Over Items (per source)
  → Code Node (normalize listing fields)
  → AI Chain (analyze listing + score deal)
  → Code Node (parse JSON response)
  → If fit score > 7 → HubSpot Create Deal
      → Telegram Alert
  → Continue loop
```

Scrapes live listings from two major business-for-sale marketplaces simultaneously. Each listing is parsed, normalized, and scored using the same acquisition criteria as the email workflow.

---

### Workflow 3 — Daily and Weekly Reports (`workflow_3_reporting.json`)

```
[Daily] Schedule Trigger (9AM daily)
  → HubSpot Search API (deals created in last 24 hours)
  → Code Node (sort by fit score, build report)
  → Telegram (send daily digest)

[Weekly] Schedule Trigger (9AM weekly)
  → HubSpot Search API (deals created in last 7 days)
  → Code Node (source breakdown, avg score, top 10)
  → Telegram (send weekly pipeline report)
```

---

## HubSpot Custom Fields Required

Before importing the workflows, create these custom properties in your HubSpot deal object:

| Field Name | Type |
|---|---|
| industry | Single-line text |
| geography | Single-line text |
| annual_revenue | Number |
| ebitda | Number |
| fit_score | Number |
| fit_reasoning | Multi-line text |
| ai_summary | Multi-line text |
| deal_source | Single-line text |
| listing_url | Single-line text |

---

## Setup Instructions

### 1. Clone the Repository

```bash
git clone https://github.com/your-username/n8n-ai-deal-sourcing-crm.git
```

### 2. Import Workflows into n8n

- Open your n8n instance
- Go to Workflows → Import from File
- Import each JSON file one by one

### 3. Connect Credentials

| Credential | Used In |
|---|---|
| Gmail OAuth2 | Workflow 1 |
| OpenRouter API | Workflow 1 and 2 |
| HubSpot App Token | Workflow 1 and 2 |
| Apify API Token | Workflow 2 |
| Telegram Bot API | Workflow 1, 2, and 3 |

### 4. Set Up HubSpot

- Create the custom deal properties listed above
- Create a pipeline with deal stages that match your acquisition process
- Copy your HubSpot Portal ID and update the Telegram alert links in the workflows

### 5. Set Up Telegram Bot

- Create a bot via BotFather on Telegram
- Get your bot token and your personal chat ID
- Update the Telegram nodes in all three workflows with your chat ID

### 6. Configure Apify

- Create an Apify account
- Get your API token
- The scrapers used are `crawlerbros/bizbuysell-scraper` and `parseforge/bizquest-scraper`
- Update search filters (industry, location, maxItems) inside the HTTP Request nodes in Workflow 2

### 7. Activate Workflows

- Activate Workflow 2 first (marketplace scraper)
- Then activate Workflow 1 (email intake)
- Then activate Workflow 3 (reporting)

---

## Folder Structure

```
n8n-ai-deal-sourcing-crm/
├── README.md
├── workflows/
│   ├── workflow_1_email_intake.json
│   ├── workflow_2_marketplace_scraper.json
│   └── workflow_3_reporting.json
```

---

## Author

**Abu Bakkar Siddik**
