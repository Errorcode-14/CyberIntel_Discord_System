# CyberIntel Discord Bot 🛡️

A free, fully automated cybersecurity intelligence bot that watches the internet for you and drops everything worth knowing straight into your Discord server — CVEs, breaches, exploits, threat research, new tools, and more.


---

## How it works

The bot wakes up 5 times a day on GitHub Actions, pulls from 20+ sources across RSS feeds, public APIs, and web scraping, deduplicates everything so you never see the same article twice, and posts rich embed cards into your Discord channels. Each card shows the article title, a real summary pulled from the page, a clickable URL, and a native Discord link preview below it.

---

## Your channels and what flows into them

### `@threat-intel` — Deep threat intelligence
The serious stuff. APT campaigns, nation-state activity, breach post-mortems, and in-depth malware analysis from the most trusted names in the industry.

**RSS feeds:**
- Krebs on Security
- Schneier on Security
- ESET Research
- Check Point Research
- WeLiveSecurity

**Scraped (no RSS):**
- Mandiant Blog
- Unit42 by Palo Alto Networks
- Recorded Future Blog
- CrowdStrike Blog
- Microsoft Security Blog

---

### `#cve-updates` — Vulnerability alerts
Every CVE published in the last 24 hours with a CVSS score of 7.0 or above. Critical CVEs (9.0+) trigger an @everyone ping. CISA-confirmed actively exploited vulnerabilities get a separate @everyone alert the moment they're added to the KEV catalog.

**Sources:**
- NVD / NIST CVE API v2.0 — scored vulnerabilities
- CISA Known Exploited Vulnerabilities (KEV) catalog

---

### `#bug-bounty` — Exploits and vulnerability research
Public PoC exploits, bug bounty disclosures, and cutting-edge vulnerability research from researchers and platforms.

**RSS feeds:**
- Exploit-DB
- HackerOne Hacktivity
- PortSwigger Research
- Chromium Bug Tracker
- Intigriti Blog
- Medium BugBountyWriteup

**Scraped (no RSS):**
- Google Project Zero
- watchTowr Labs
- Zero Day Initiative (ZDI)

---

### `#daily-news` — Security news digest
The daily news cycle — breaches, ransomware, industry news, government advisories, and everything else worth skimming.

**RSS feeds:**
- Bleeping Computer
- The Hacker News
- SANS ISC Diary
- Dark Reading
- SecurityWeek
- Threatpost
- CIS Advisories
- Cyber Security News

**Scraped (no RSS):**
- Help Net Security

---

### `#tools-resources` — New tools and research
Newly created security repositories on GitHub across 4 topic areas, plus the latest academic security research from ArXiv.

**Sources:**
- GitHub API — new repos tagged: `cybersecurity`, `penetration-testing`, `osint`, `bug-bounty`
- ArXiv cs.CR — cryptography and security research papers

---

## Setup

### 1. Fork or clone this repo

```bash
git clone https://github.com/YOUR_USERNAME/cyber-intel-bot.git
cd cyber-intel-bot
```

### 2. Create Discord webhooks

For each of your 5 channels, do this:
1. Right-click the channel → **Edit Channel**
2. Go to **Integrations → Webhooks → New Webhook**
3. Name it `CyberIntel Bot`
4. Click **Copy Webhook URL** and save it

### 3. Add secrets to GitHub

Go to your repo → **Settings → Secrets and variables → Actions → New repository secret**

Add these 5 secrets:

| Secret name | Webhook for |
|---|---|
| `DISCORD_THREAT_INTEL` | `@threat-intel` |
| `DISCORD_CVE_UPDATES` | `#cve-updates` |
| `DISCORD_BUG_BOUNTY` | `#bug-bounty` |
| `DISCORD_DAILY_NEWS` | `#daily-news` |
| `DISCORD_TOOLS_RESOURCES` | `#tools-resources` |

> `GITHUB_TOKEN` is automatically provided by GitHub Actions — you don't need to add it.

### 4. Test it

Go to **Actions tab → CyberIntel Discord Bot → Run workflow**

First run takes about 3 minutes. Check your Discord — everything should start coming in.

---

## Run schedule (IST)

| Time | What runs |
|---|---|
| 6:30 AM | CVE check + threat intel |
| 9:30 AM | Daily news digest |
| 1:30 PM | Full sweep — all channels |
| 5:30 PM | Bug bounty + tools |
| 9:30 PM | Research papers + wrap-up |

---

## Adding a new source

### Site has RSS — add one line

Find the feed list for the channel you want in `src/bot.py` and add your site:

```python
DAILY_NEWS_FEEDS = [
    # ... existing feeds ...
    ("https://yoursite.com/feed/", "Your Site Name"),  # add here
]
```

Feed list → channel mapping:
- `DAILY_NEWS_FEEDS` → `#daily-news`
- `THREAT_INTEL_FEEDS` → `@threat-intel`
- `BUG_BOUNTY_FEEDS` → `#bug-bounty`

### Site has no RSS — add a scrape block

**Step 1:** Open the site in Chrome, press F12, click the arrow icon, hover over an article card to find:
- The repeating wrapper element (e.g. `article`, `div.post-card`)
- The title element inside it (e.g. `h2`, `h3`)
- The link element (e.g. `a`, `h3 a`)

**Step 2:** Add it to the right function in `src/bot.py`:

```python
def fetch_daily_news_scraped() -> list:
    items = []
    # ... existing scrape blocks ...

    items += scrape(
        url         = "https://example-security-blog.com/",
        article_sel = "article.post-card",   # repeating wrapper
        title_sel   = "h2",                  # title element
        link_sel    = "a",                   # link element
        base_url    = "https://example-security-blog.com",
        label       = "Example Blog",
        webhook_key = "daily_news",
        color       = "news",
    )
    return items
```

Function → channel mapping:
- `fetch_daily_news_scraped()` → `#daily-news`
- `fetch_threat_intel_scraped()` → `@threat-intel`
- `fetch_bug_bounty_scraped()` → `#bug-bounty`

**Common CSS selector patterns that work on most security blogs:**

```python
# WordPress (most common)
article_sel = ".post, article"
title_sel   = ".entry-title, h2.post-title"
link_sel    = ".entry-title a, h2 a"

# Card-based layouts
article_sel = ".card, .blog-card, .post-card"
title_sel   = "h2, h3, .card-title"
link_sel    = "a"

# Simple article tags
article_sel = "article"
title_sel   = "h2, h3"
link_sel    = "a"
```

If the scraper returns 0 items, the site is likely JavaScript-rendered (Cloudflare protected). Check if they have an RSS feed instead — most do.

### After adding anything

```bash
git add src/bot.py
git commit -m "add: Site Name as source"
git push
```

Changes take effect on the next scheduled run, or trigger it manually from the Actions tab.

---

## Customisation

**Only want critical CVEs (CVSS ≥ 9.0):**
```python
# src/bot.py — in the run() function
all_items += fetch_nvd_cves(min_cvss=9.0, hours=24)  # change 7.0 to 9.0
```

**Filter by keyword — only post if article matches:**
```python
# add this inside run(), just before the send_embed call
KEYWORDS = ["ransomware", "zero-day", "rce", "actively exploited", "critical"]
if not any(k in item["title"].lower() for k in KEYWORDS):
    skipped += 1
    continue
```

**Change the run schedule:**
Edit the cron lines in `.github/workflows/cyber_intel.yml`. Use [crontab.guru](https://crontab.guru) to build a custom schedule. Times are in UTC — subtract 5:30 for IST.

---

## How deduplication works

Every time the bot posts something, it saves a fingerprint of that item to `sent_ids.json`. GitHub Actions caches this file between runs using `actions/cache`. So even if an article stays in a feed for a week, it only ever gets posted to Discord once. The cache holds the last 2,000 items.

---

## Troubleshooting

Check the logs first: **Actions tab → click the latest run → click `run-bot`**

| Log line | What it means | Fix |
|---|---|---|
| `✗ Failed` | Discord rejected the post | Check the webhook URL is correct in secrets |
| `Webhook not configured` | Secret not set | Add the missing secret in GitHub Settings |
| `Scraped X: 0 items` | Selectors didn't match or site uses JS | Re-inspect the HTML or switch to RSS |
| `403 Client Error` | Site blocked the scraper (Cloudflare) | Use their RSS feed instead |
| `Rate limited` | Too many Discord posts at once | Bot handles this automatically — no action needed |

---

## Cost

**$0.** GitHub Actions free tier gives 2,000 minutes/month. This bot uses roughly 300 minutes/month (5 runs/day × ~2 min each). You have plenty of headroom.

---

## File structure

```
cyber-intel-bot/
├── .github/
│   └── workflows/
│       └── cyber_intel.yml   ← schedule + secrets config
├── src/
│   └── bot.py                ← all the logic lives here
├── requirements.txt           ← requests, feedparser, beautifulsoup4, lxml
├── .gitignore
└── README.md
```
