# Micro-influencer-outreach-automation

**Niche:** Fitness & Beauty | **Platform:** Instagram | **Region:** India | **Influencers:** 50

---

## What This Is

This project automates the entire process of discovering Indian micro-influencers, enriching their profiles, classifying them into collaboration segments, generating personalized outreach messages using AI, and sending emails — all without any manual work after the initial setup.

The system is built on **n8n**, an open-source workflow automation tool, and uses Google Gemini for AI message generation and Gmail API for sending outreach emails.

---

## Two Approaches — Here's the Honest Story

While building this, I took two approaches, and I want to be transparent about both.

### Approach 1 — API-Based Scraping (workflow_api.json)

The original plan was to scrape Instagram in real time using the **Apify Instagram Hashtag Scraper**. The workflow would:

1. Take hashtags like `#fitnessindia`, `#indianbeautyblogger`
2. Scrape recent posts and extract profile usernames
3. Run those usernames through the Apify Profile Scraper to get follower counts, bios, and engagement data
4. Feed that into the enrichment and outreach pipeline

This workflow is fully built and the architecture is production-ready. The problem I ran into was Apify's free plan limits — 25 concurrent actor runs maximum, and the hashtag results kept returning the same cached accounts rather than diverse profiles. This is a known limitation of free-tier scraping tools; Instagram actively blocks scraping at scale.

The workflow would work correctly with a paid Apify plan or a more robust scraping API.

### Approach 2 — Curated Sample Dataset (workflow_sample.json)

To validate that the full pipeline actually works — the filtering, enrichment, segmentation, AI generation, email sending, and Google Sheets logging — I replaced the Apify scraping nodes with a JavaScript Code node that outputs a curated dataset of 50 real Indian fitness and beauty micro-influencers.

This approach successfully executed the complete pipeline end to end. Emails were sent, Google Sheets were filled with all columns, and Gemini generated personalized messages for each influencer.

The sample data is not fake — it consists of real Indian influencers sourced from Qoruz, Modash, Feedspot, and manual Instagram research. The difference is that I loaded the data directly instead of scraping it live.

---

## Influencer Discovery

**Sources used:**
- Qoruz and Modash (Indian influencer databases)
- Feedspot's Top Indian Fitness and Beauty Instagram lists
- Manual hashtag research on Instagram

**Niches covered:** Fitness and Beauty
**Geography:** India
**Follower range:** 5,000 to 100,000 (micro-influencer definition)

Some of the real influencers included in the dataset: Yasmin Karachiwala, Namrata Purohit, Simrun Chopra, Luke Coutinho, Komal Pandey, Kritika Khurana, Debasree Banerjee, Aashna Shroff, and others.

---

## How the Workflow Works

### Step 1 — Trigger

The workflow runs on a weekly schedule every Monday at 9:00 AM via n8n's Schedule Trigger. Fully automated, no manual action needed.

### Step 2 — Data Input

In the API workflow, this is the Apify scraping chain. In the sample workflow, a JavaScript Code node outputs all 50 influencer profiles directly. Either way, the output is the same — a list of Instagram profiles with username, followers, bio, and basic metrics.

### Step 3 — Filtering

A Filter node keeps only profiles with followers between 5,000 and 100,000. Anyone outside that range is dropped automatically.

### Step 4 — Profile Enrichment

A JavaScript Code node processes each profile and computes:

- **Engagement rate** — calculated as `(average likes + average comments) / followers × 100`
- **Contact email** — extracted from the bio using regex
- **City** — detected by matching bio text against major Indian cities (Mumbai, Delhi, Bangalore, Chennai, Hyderabad, Pune, Kolkata, etc.)
- **Content themes** — classified from bio keywords into categories like Yoga, Workout, Nutrition, Skincare, Mental Wellness, Running

### Step 5 — Segmentation

A Switch node classifies each influencer into one of three segments:

| Segment | Criteria | Collaboration Type |
|---------|----------|--------------------|
| Segment A — Nano Creators | Followers up to 30,000 | Barter / UGC Creation |
| Segment B — Mid Tier Micro | Followers 30,001 to 100,000 | Paid Sponsorship / Affiliate |
| Segment C — High Potential | Engagement rate above 5% | Brand Ambassador |

Each segment tag node adds the segment label and collaboration type to the profile while keeping all existing data intact.

### Step 6 — AI Message Generation

Each profile is sent to **Google Gemini 1.5 Flash** via n8n's LangChain integration. The prompt passes the influencer's name, niche, content themes, engagement rate, city, segment, and bio to generate two messages:

- An **email pitch** of 60 to 90 words with a professional tone, referencing their specific niche and content style
- An **Instagram DM** of 15 to 30 words, casual and direct

The AI response comes back as JSON, which a Parse node extracts and merges back with the full profile data.

A 60-second wait between loop iterations prevents hitting Gemini's API rate limits.

### Step 7 — Sending Outreach

If a contact email was found in the bio, Gmail sends a personalized collaboration email using the Gmail API (OAuth2). The subject line includes the influencer's name, and the body is the AI-generated pitch.

For Instagram DMs — the generated message is stored in Google Sheets. Instagram's API restricts automated DMs for non-business accounts, so these are ready for manual or tool-assisted sending.

### Step 8 — Logging to Google Sheets

Two Google Sheets are updated automatically after each influencer is processed:

**Raw Data Sheet** — every influencer with their full enriched profile, AI-generated messages, and email sent status.

**Segments Sheet** — same data, used for segment-specific tracking.

After an email is sent, the workflow goes back and updates the `Email Sent` column to `Yes` for that row, matched by the profile link.

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
| n8n (self-hosted) | Workflow automation |
| Google Gemini 1.5 Flash | AI message generation |
| Gmail API | Sending outreach emails |
| Google Sheets API | Data logging |
| Apify | Instagram scraping (API workflow) |
| JavaScript | Enrichment logic and parsing |

---

## Repository Structure

```
assignment-4-influencer-outreach/
│
├── README.md               — This file
├── workflow_api.json       — Apify-based scraping workflow
├── workflow_sample.json    — Sample data workflow (fully executed)
└── influencers.csv         — 50 real Indian influencers dataset
```

### How to Run the Sample Workflow

1. Import `workflow_sample.json` into your n8n instance
2. Set up credentials for Google Sheets, Gmail, and Google Gemini
3. Execute the workflow manually or wait for the Monday trigger
4. Check your Google Sheets for the output

---

