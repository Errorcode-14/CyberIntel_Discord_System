# CyberIntel Discord Bot 🛡️

Pushes cybersecurity intelligence into your 5 Discord channels automatically.
Runs free on GitHub Actions — no server, no cost.

---

## Your channels → what goes where

| Discord channel   | Secret name                   | What gets posted |
|-------------------|-------------------------------|-----------------|
| `@threat-intel`   | `DISCORD_THREAT_INTEL`        | APT reports, breach analysis, Krebs, Mandiant, Unit42, CrowdStrike, Recorded Future |
| `#cve-updates`    | `DISCORD_CVE_UPDATES`         | All CVEs CVSS ≥ 7.0 from NVD + CISA actively-exploited list |
| `#bug-bounty`     | `DISCORD_BUG_BOUNTY`          | Exploit-DB PoCs, HackerOne disclosures, Project Zero, ZDI, watchTowr |
| `#daily-news`     | `DISCORD_DAILY_NEWS`          | BleepingComputer, The Hacker News, Dark Reading, SANS ISC, SecurityWeek |
| `#tools-resources`| `DISCORD_TOOLS_RESOURCES`     | GitHub new security tools, ArXiv research papers |

---

## Setup (5 minutes)

### Step 1 — Push to GitHub
```bash
git init
git add .
git commit -m "initial commit"
git remote add origin https://github.com/YOUR_USERNAME/cyber-intel-bot.git
git push -u origin main
```

### Step 2 — Create Discord webhooks
For **each of your 5 channels**:
1. Right-click the channel → **Edit Channel**
2. **Integrations** → **Webhooks** → **New Webhook**
3. Name it `CyberIntel Bot`, click **Copy Webhook URL**

### Step 3 — Add secrets to GitHub
Go to your repo → **Settings → Secrets and variables → Actions → New repository secret**

Add exactly these 5 secrets:

| Secret name | Webhook URL from |
|---|---|
| `DISCORD_THREAT_INTEL` | `@threat-intel` webhook |
| `DISCORD_CVE_UPDATES` | `#cve-updates` webhook |
| `DISCORD_BUG_BOUNTY` | `#bug-bounty` webhook |
| `DISCORD_DAILY_NEWS` | `#daily-news` webhook |
| `DISCORD_TOOLS_RESOURCES` | `#tools-resources` webhook |

> `GITHUB_TOKEN` is auto-provided — no setup needed.

### Step 4 — Test it now
Go to your repo → **Actions** tab → **CyberIntel Discord Bot** → **Run workflow**

---

## Run schedule (IST)

| IST time | What runs |
|---|---|
| 06:30 | CVE check + threat intel |
| 09:30 | Daily news digest |
| 13:30 | Full sweep (all channels) |
| 17:30 | Bug bounty + tools |
| 21:30 | Research papers + wrap-up |

---

## Customise

**Raise CVE threshold** (fewer, more severe only):
```python
# in src/bot.py line ~130
all_items += fetch_nvd_cves(min_cvss=9.0, hours=24)  # critical only
```

**Add a new RSS feed to #daily-news:**
```python
DAILY_NEWS_FEEDS = [
    ...
    ("https://your-site.com/feed/", "Your Site Label"),
]
```

**Add a keyword filter** (only post if article matches):
```python
KEYWORDS = ["ransomware", "zero-day", "rce", "actively exploited"]
if not any(k in item["title"].lower() for k in KEYWORDS):
    continue
```

---

## Cost: $0
GitHub Actions free tier = 2,000 min/month.
This bot uses ~1–2 min × 5 runs/day = ~300 min/month.
