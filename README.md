# Automated Micro-Influencer Outreach System

**Niche:** Fitness & Beauty &nbsp;|&nbsp; **Platform:** Instagram &nbsp;|&nbsp; **Region:** India &nbsp;|&nbsp; **Influencers:** 50

---

## What This Is

This project automates the entire process of discovering Indian micro-influencers, enriching their profiles, classifying them into collaboration segments, generating personalized outreach messages using AI, and sending emails — all without any manual work after the initial setup.

The system is built on **n8n** (open-source workflow automation), uses **Google Gemini 1.5 Flash** for AI message generation, and **Gmail API** for sending outreach emails. It runs on a weekly schedule with zero human intervention.

---



## Two Approaches — The Honest Story

While building this, I took two approaches and want to be transparent about both.

### Approach 1 — API-Based Scraping (`workflow_api.json`)

The original plan was to scrape Instagram in real time using the **Apify Instagram Hashtag Scraper**. The workflow would:

1. Take hashtags like `#fitnessindia`, `#indianbeautyblogger`
2. Scrape recent posts and extract profile usernames
3. Run those through Apify's Profile Scraper to get follower counts, bios, and engagement data
4. Feed everything into the enrichment and outreach pipeline

This workflow is fully built and production-ready in architecture. The issue was Apify's free plan — 25 concurrent actor runs max, and hashtag results kept returning the same cached accounts. This is a known limitation of free-tier scraping; Instagram actively blocks scraping at scale. With a paid Apify plan or a stronger API, this workflow runs as intended.

### Approach 2 — Curated Sample Dataset (`workflow_sample.json`)

To prove the full pipeline actually works — filtering, enrichment, segmentation, AI generation, email sending, and Sheets logging — I replaced the Apify nodes with a JavaScript Code node that directly outputs a curated dataset of 50 real Indian influencers.

This ran end to end successfully. Emails were sent, Google Sheets filled up with all columns, and Gemini generated personalized messages for every influencer.

The data is not made up — these are real Indian influencers sourced from Qoruz, Modash, Feedspot, and manual Instagram research. The only difference is the data was loaded directly rather than scraped live.

---

## Influencer Discovery

| Detail | Info |
|--------|------|
| Sources | Qoruz, Modash, Feedspot, Instagram hashtag research |
| Niches | Fitness and Beauty |
| Geography | India |
| Follower Range | 5,000 to 100,000 |

Some real influencers in the dataset: Yasmin Karachiwala, Namrata Purohit, Simrun Chopra, Luke Coutinho, Komal Pandey, Kritika Khurana, Debasree Banerjee, Aashna Shroff, and more.

---

## How the Workflow Works

### Step 1 — Trigger
Runs automatically every Monday at 9:00 AM via n8n's Schedule Trigger. No manual action needed.

### Step 2 — Data Input
In the API workflow, this is the Apify scraping chain. In the sample workflow, a JavaScript Code node outputs all 50 profiles directly. Either way the output is identical — a list of Instagram profiles with username, followers, bio, and metrics.

### Step 3 — Filtering
A Filter node keeps only profiles with followers between 5,000 and 100,000. Everything outside that range is dropped.

### Step 4 — Profile Enrichment
A JavaScript Code node enriches each profile with:

- **Engagement rate** — `(avg likes + avg comments) / followers × 100`
- **Contact email** — regex extraction from bio text
- **City** — keyword matching against major Indian cities in the bio
- **Content themes** — bio keyword classification into Yoga, Workout, Nutrition, Skincare, Mental Wellness, Running, etc.

### Step 5 — Segmentation
A Switch node classifies each influencer into one of three segments:

| Segment | Criteria | Collaboration Type |
|---------|----------|--------------------|
| Segment A — Nano Creators | Followers up to 30,000 | Barter / UGC Creation |
| Segment B — Mid Tier Micro | Followers 30,001 to 100,000 | Paid Sponsorship / Affiliate |
| Segment C — High Potential | Engagement rate above 5% | Brand Ambassador |

### Step 6 — AI Message Generation
Each profile is sent to **Google Gemini 1.5 Flash** with a structured prompt that generates:

- An **email pitch** — 60 to 90 words, professional tone, references niche and content style
- An **Instagram DM** — 15 to 30 words, casual and direct

The response comes back as JSON, which a Parse node extracts and merges back with the full profile. A 60-second wait between iterations prevents Gemini API rate limit errors.

### Step 7 — Outreach Sending
If a contact email was found in the bio, Gmail sends the personalized pitch via Gmail API (OAuth2). Subject includes the influencer's name, body is the AI-generated email pitch.

Instagram DMs are stored in Google Sheets for manual or tool-assisted sending — Instagram's API restricts automated DMs for non-business accounts.

### Step 8 — Google Sheets Logging
Two sheets are updated after each influencer is processed:

- **Raw Data Sheet** — full profile, AI messages, and email sent status
- **Segments Sheet** — same data for segment-specific tracking

Once an email is sent, the `Email Sent` column is updated to `Yes`, matched by profile link.

---

## Sample Output

**Email Pitch — Yasmin Karachiwala (Segment B)**

> Dear Yasmin, we have been following your work for a while and are genuinely impressed by how consistently you bring real fitness knowledge to your audience. As an Indian wellness brand, we think your approach to mindful movement aligns closely with what we stand for. We would love to explore a paid sponsorship that adds genuine value to your content. Would you be open to a conversation?

**Instagram DM — Komal Pandey (Segment C)**

> Hi Komal! Love your bold take on beauty and styling. We are an Indian wellness brand and would love to collaborate. Let us connect!

---

## Tech Stack

| Tool | Purpose |
|------|---------|
| n8n (self-hosted) | Workflow automation engine |
| Google Gemini 1.5 Flash | AI message generation |
| Gmail API | Sending outreach emails |
| Google Sheets API | Data logging |
| Apify | Instagram scraping (API workflow) |
| JavaScript | Enrichment logic and response parsing |

---

## Repository Structure

```
micro-influencer-outreach-automation/
│
├── README.md                    — This file
├── workflow_api.json            — Apify-based scraping workflow
├── workflow_sample.json         — Sample data workflow (fully executed)
├── influencers.csv              — 50 real Indian influencers dataset
└── screenshots/
    ├── workflow_overview.png
    ├── google_sheets_output.png
    ├── gmail_sent.png
    └── ai_messages.png
```

### How to Run

1. Import `workflow_sample.json` into your n8n instance
2. Set up credentials for Google Sheets, Gmail, and Google Gemini
3. Execute manually or wait for the Monday 9AM trigger
4. Check your Google Sheets for the output

---
