# CyberIntel Discord System 🛡️

A scheduled cybersecurity intelligence pipeline that aggregates threat data from RSS feeds, APIs, and web scraping, then distributes structured alerts into five categorized Discord channels.

No server needed. No paid subscriptions. Just push to GitHub and it runs itself.

---

## 🎬 Demo

> Video coming soon — will showcase live intelligence posts flowing into each Discord channel.


       [Demo](https://github.com/Errorcode-14/CyberIntel_Discord_System/blob/main/src/Demo.mp4)

---

## 📡 Intelligence Streams

Join the public Discord server to see the system in action:

**[Join the Discord Server →](https://discord.gg/fXgMpvcsyp)**

| Channel | What you'll see |
|---|---|
| 📰 `#daily-news` | Daily security news digest |
| 🚨 `#cve-updates` | CVE alerts — CVSS 3.0 to 10.0, color coded by severity |
| 🎯 `@threat-intel` | APT reports, breach analysis, threat intelligence |
| 🐞 `#bug-bounty` | Public exploits, PoCs, and vulnerability research |
| 🛠️ `#tools-resources` | New security tools and research papers |

Updates are automatically collected, categorized, and published multiple times per day from 20+ trusted cybersecurity sources.

---

## How it works

The bot wakes up 5 times a day on GitHub Actions, pulls from 20+ sources across RSS feeds, public APIs, and web scraping, deduplicates everything so you never see the same article twice, and posts rich embed cards into your Discord channels. Each card shows a color-coded severity, the article title, a real summary, a clickable URL, and a native Discord link preview below it.

---

## CVE Severity Color Coding

All CVEs from CVSS 3.0 and above are posted — color coded by severity so you can tell at a glance what needs immediate attention:

| Color | Emoji | Severity | CVSS Range |
|---|---|---|---|
| 🔴 Red | 🔴 | CRITICAL | 9.0 – 10.0 |
| 🟠 Orange | 🟠 | HIGH | 7.0 – 8.9 |
| 🟡 Yellow | 🟡 | MEDIUM | 4.0 – 6.9 |
| 🔵 Blue | 🔵 | LOW | 3.0 – 3.9 |
| ⚪ Grey | ⚪ | PENDING | Score not yet assigned |

Critical CVEs (9.0+) trigger an **@everyone** ping. All others post silently.

---

## Your channels and what flows into them

### `@threat-intel` — Deep threat intelligence
APT campaigns, nation-state activity, breach post-mortems, and in-depth malware analysis.

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

### `#cve-updates` — Vulnerability alerts (CVSS 3.0–10.0)
Every CVE published in the last 24 hours with a CVSS score of 3.0 or above, sorted by severity. CISA-confirmed actively exploited vulnerabilities get a separate @everyone alert the moment they hit the KEV catalog.

**Primary source:**
- NVD / NIST CVE API v2.0 (with free API key for reliability)

**Automatic fallback when NVD is down:**
- CVEProject/cvelistV5 - official MITRE CVE list on GitHub (delta zip, updated every few hours)
- Packet Storm vulnerabilities feed

**Always running alongside:**
- CISA Known Exploited Vulnerabilities (KEV) catalog

---

### `#bug-bounty` — Exploits and vulnerability research
Public PoC exploits, bug bounty disclosures, and cutting-edge vulnerability research.

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
The daily news cycle — breaches, ransomware, industry news, government advisories.

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
Newly created security repositories on GitHub + latest academic security research from ArXiv.

**Sources:**
- GitHub API — new repos tagged: `cybersecurity`, `penetration-testing`, `osint`, `bug-bounty`
- ArXiv cs.CR — cryptography and security research papers

---

## Setup

### 1. Fork or clone this repo

```bash
git clone https://github.com/Errorcode-14/CyberIntel_Discord_System.git
cd CyberIntel_Discord_System
```

### 2. Get a free NVD API key (highly recommended)

Without a key, the NVD API is rate-limited and often returns empty responses.
The bot has a fallback (CVEProject GitHub) but the key keeps NVD reliable.

1. Go to **https://nvd.nist.gov/developers/request-an-api-key**
2. Enter your email — key arrives instantly in your inbox
3. Add it as a GitHub secret named `NVD_API_KEY` (see step 4)

### 3. Create Discord webhooks

For each of your 5 channels:
1. Right-click the channel → **Edit Channel**
2. Go to **Integrations → Webhooks → New Webhook**
3. Name it `CyberIntel name*`
4. Click **Copy Webhook URL**

### 4. Add secrets to GitHub

Go to your repo → **Settings → Secrets and variables → Actions → New repository secret**

| Secret name | Value |
|---|---|
| `DISCORD_THREAT_INTEL` | Webhook URL for `@threat-intel` |
| `DISCORD_CVE_UPDATES` | Webhook URL for `#cve-updates` |
| `DISCORD_BUG_BOUNTY` | Webhook URL for `#bug-bounty` |
| `DISCORD_DAILY_NEWS` | Webhook URL for `#daily-news` |
| `DISCORD_TOOLS_RESOURCES` | Webhook URL for `#tools-resources` |
| `NVD_API_KEY` | Your NVD API key (free) |

> `GITHUB_TOKEN` is automatically provided by GitHub Actions — you don't need to add it.

### 5. Test it

Go to **Actions tab → CyberIntel Discord System → Run workflow**

First run takes about 5 minutes. Check your Discord - everything should start flowing in.

---

## Run Schedule (IST)

> ⚠️ GitHub Actions free tier does not guarantee exact run times. Actual runs may be delayed by 1–4 hours depending on GitHub's server load. This is normal and outside our control.

| Scheduled (IST) | Targets                   | Cron (UTC)              |
| --------------- | ------------------------- | ----------------------- |
| ~6:30 AM        | CVE check + threat intel  | `0 1 * * *` (prev. day) |
| ~9:30 AM        | Daily news digest         | `0 4 * * *`             |
| ~1:30 PM        | Full sweep — all channels | `0 8 * * *`             |
| ~5:30 PM        | Bug bounty + tools        | `0 12 * * *`            |
| ~9:30 PM        | Research papers + wrap-up | `0 16 * * *`            |

---

## Adding a new source

### Site has RSS — add one line

Find the right feed list in `src/bot.py` and add your site:

```python
DAILY_NEWS_FEEDS = [
    # ... existing feeds ...
    ("https://yoursite.com/feed/", "Your Site Name"),
]
```

Feed list → channel mapping:
- `DAILY_NEWS_FEEDS` → `#daily-news`
- `THREAT_INTEL_FEEDS` → `@threat-intel`
- `BUG_BOUNTY_FEEDS` → `#bug-bounty`

### Site has no RSS — scraping

**Step 1:** Open the site in Chrome, press F12, click the arrow icon, hover over an article card to find the repeating wrapper, title element, and link element.

**Step 2:** Add a scrape block to the right function:

```python
def fetch_daily_news_scraped() -> list:
    items = []
    # ... existing blocks ...

    items += scrape(
        url         = "https://example.com/news/",
        article_sel = "article",          # repeating wrapper
        title_sel   = "h2",               # title element
        link_sel    = "a",                # link element
        base_url    = "https://example.com",
        label       = "Example Blog",
        webhook_key = "daily_news",
        color       = "news",
    )
    return items
```

Function → channel:
- `fetch_daily_news_scraped()` → `#daily-news`
- `fetch_threat_intel_scraped()` → `@threat-intel`
- `fetch_bug_bounty_scraped()` → `#bug-bounty`

**Common CSS patterns that work on most blogs:**

```python
# WordPress
article_sel = ".post, article"
title_sel   = ".entry-title, h2.post-title"
link_sel    = ".entry-title a"

# Card-based layouts
article_sel = ".card, .blog-card, .post-card"
title_sel   = "h2, h3, .card-title"
link_sel    = "a"

# Year-based archive pages (e.g. Help Net Security)
url = f"https://www.helpnetsecurity.com/{datetime.now().year}/"
```

> If scraping returns 0 items, the site is likely JavaScript-rendered (Cloudflare). Check if they have an RSS feed instead - most do.

---

## Customisation

**Change the CVSS threshold** — edit in `run()`:
```python
# Show only HIGH and above (7.0+)
nvd_items = fetch_nvd_cves(min_cvss=7.0, hours=24)

# Show everything from LOW (3.0+) — current default
nvd_items = fetch_nvd_cves(min_cvss=3.0, hours=24)
```

**Filter by keyword:**
```python
# Add inside run(), just before the send_embed call
KEYWORDS = ["ransomware", "zero-day", "rce", "actively exploited"]
if not any(k in item["title"].lower() for k in KEYWORDS):
    skipped += 1
    continue
```

**Change the schedule:**
Edit cron lines in `.github/workflows/cyber_intel.yml`. Use [crontab.guru](https://crontab.guru) to build a custom schedule. Times are UTC - subtract 5:30 for IST.

---

## How deduplication works

Every time the bot posts something, it saves a fingerprint of that item to `sent_ids.json`. GitHub Actions caches this file between runs using `actions/cache`. So even if an article stays in a feed for a week, it only ever gets posted once. Cache holds the last 2,000 items.

---

## Troubleshooting

Check logs: **Actions tab → click the latest run → click `run-bot`**

| Log line | Meaning | Fix |
|---|---|---|
| `NVD empty response` | NVD API is down or rate-limited | Bot auto-switches to CVEProject fallback |
| `CVEProject GitHub failed: name 'zipfile' is not defined` | Missing imports in your file | Add `import io` and `import zipfile` at the top of `bot.py` |
| `✗ Failed` | Discord rejected the post | Check webhook URL is correct in secrets |
| `Webhook not configured` | Secret not set | Add the missing secret in GitHub Settings |
| `Scraped X: 0 items` | Selectors didn't match or site uses JS | Re-inspect HTML or switch to RSS |
| `403 Client Error` | Site blocked scraper (Cloudflare) | Use their RSS feed instead |
| `Rate limited` | Too many Discord posts at once | Bot handles this automatically |

---

## Cost

**$0.** GitHub Actions free tier gives 2,000 minutes/month. This bot uses roughly 900 minutes/month (5 runs/day × ~6 min each).

---

## File structure

```
CyberIntel_Discord_System/
├── .github/
│   └── workflows/
│       └── cyber_intel.yml   ← schedule, secrets, Node.js 24
├── src/
│   └── bot.py                ← all logic lives here
├── requirements.txt           ← requests, feedparser, beautifulsoup4, lxml
├── .gitignore
└── README.md
```
