# 🚀 Running Mirror-Leech-Telegram-Bot on Google Colab

This guide walks you through deploying and running **Mirror-Leech-Telegram-Bot (MLTB)** directly on Google Colab without needing a paid Virtual Private Server (VPS) or local Docker installation.

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/aminelazz/mirror-leech-telegram-bot/blob/master/MLTB_Colab.ipynb)

---

## 📋 Table of Contents
1. [Overview](#-overview)
2. [Prerequisites](#-prerequisites)
3. [Quick Start Guide](#-quick-start-guide)
4. [Detailed Cell Breakdown](#-detailed-cell-breakdown)
5. [Anti-Seeding Configuration (Zero Upload)](#-anti-seeding-configuration-zero-upload)
6. [Configuring Cloudflare Tunnel (Torrent Web Selection)](#-configuring-cloudflare-tunnel-torrent-web-selection)
7. [Persisting Credentials on Google Drive](#-persisting-credentials-on-google-drive)
8. [Tips to Prevent Colab Disconnections](#-tips-to-prevent-colab-disconnections)
9. [Troubleshooting & FAQs](#-troubleshooting--faqs)

---

## 🌟 Overview

The `MLTB_Colab.ipynb` notebook provides a complete runtime environment in Google Colab that:
- Installs all Linux system binaries: `aria2c`, `qbittorrent-nox`, `sabnzbdplus`, `ffmpeg`, `p7zip`, `par2`, and `rclone`.
- Automatically maps filesystem paths (`/app` and `/usr/src/app`) to the Colab workspace.
- Provides an **interactive form UI** for configuring all Telegram and bot settings without manually editing files.
- Supports **Google Drive mounting** to persist your `config.py`, `token.pickle`, `rclone.conf`, and Service Accounts (`accounts.zip`) across session restarts.
- Integrates optional **Cloudflare Tunnel** (`cloudflared`) to expose `BASE_URL` with a secure public HTTPS link for torrent and NZB web file selection.
- Isolates event loops and streams logs live with clean start/stop handlers.

---

## 🔑 Prerequisites

Before launching the notebook, gather the following:

| Parameter | Description | Where to Get |
| :--- | :--- | :--- |
| `BOT_TOKEN` | Your Telegram Bot token | [@BotFather](https://t.me/BotFather) |
| `OWNER_ID` | Your numeric Telegram User ID | [@userinfobot](https://t.me/userinfobot) |
| `TELEGRAM_API` | Telegram API App ID (`api_id`) | [my.telegram.org](https://my.telegram.org) |
| `TELEGRAM_HASH` | Telegram API App Hash (`api_hash`) | [my.telegram.org](https://my.telegram.org) |
| `DATABASE_URL` *(Optional)* | MongoDB connection string | [MongoDB Atlas](https://www.mongodb.com/atlas) (Free M0 cluster) |
| `USER_SESSION_STRING` *(Optional)* | Pyrogram session string for 4GB uploads | Run `generate_string_session.py` |
| `token.pickle` *(Optional)* | Google Drive OAuth token | Run `generate_drive_token.py` |
| `rclone.conf` *(Optional)* | Rclone remotes configuration | Generate via `rclone config` |

---

## ⚡ Quick Start Guide

1. **Open the Notebook:**
   - Click the **Open in Colab** badge above or upload [`MLTB_Colab.ipynb`](file:///MLTB_Colab.ipynb) directly to [Google Colab](https://colab.research.google.com).
2. **Run Cell 1 (Anti-Timeout Keep-Alive):**
   - Initializes audio ping and DOM refresh to prevent browser background tab suspension.
3. **Run Cell 2 (Mount Google Drive - Recommended):**
   - Mounts `/content/drive` so your credentials are saved to `My Drive/MLTB/` and never lost when Colab restarts.
4. **Run Cell 3 (Install Dependencies):**
   - Installs `aria2`, `qbittorrent-nox`, `sabnzbdplus`, `ffmpeg`, `rclone`, sets up directory symlinks, and installs Python packages from `requirements.txt`.
5. **Run Cell 4 (Bot Configuration):**
   - Fill in `BOT_TOKEN`, `OWNER_ID`, `TELEGRAM_API`, and `TELEGRAM_HASH`.
   - Toggle options for Google Drive, Rclone, Leech settings, or Cloudflare Tunnel as desired.
   - Click **Run** to generate `config.py` (and auto-save a backup to Drive).
6. **Run Cell 5 (Import Private Files - Optional):**
   - Auto-copies `token.pickle`, `rclone.conf`, and `accounts.zip` from your Google Drive folder, or presents upload buttons to upload them directly.
7. **Run Cell 6 (Start Bot):**
   - Launches all background download daemons (`aria2c`, `qbittorrent-nox`, `sabnzbdplus`) and starts the bot with real-time log output!

---

## 🔍 Detailed Cell Breakdown

### Cell 1: Anti-Timeout Keep-Alive
- **Purpose:** Prevents Google Colab from idling out when you leave the browser tab in the background.
- **Bonus Console Snippet:**
  Open browser Developer Tools (`F12` -> `Console`) and run:
  ```javascript
  function ClickConnect(){
    console.log("Colab Ping"); 
    document.querySelector("colab-connect-button")?.shadowRoot?.querySelector("#connect")?.click();
  }
  setInterval(ClickConnect, 60000);
  ```

### Cell 2: Mount Google Drive
- Creates `/content/drive/MyDrive/MLTB`.
- If you place your `config.py`, `token.pickle`, `rclone.conf`, or `accounts.zip` here once, the notebook will automatically find and load them on future runs!

### Cell 3: Environment Setup & Dependencies
- Clones `mirror-leech-telegram-bot` to `/content/mirror-leech-telegram-bot` (if not already present).
- Creates critical directory mappings:
  - `ln -sfn /content/mirror-leech-telegram-bot /app`
  - `mkdir -p /app/downloads /usr/src /JDownloader`
  - `ln -sfn /app /usr/src/app`
- Normalizes script line endings (`sed -i 's/\r$//' *.sh`) to prevent `\r` interpreter errors.
- Runs `pip install --no-cache-dir -r requirements.txt`.

### Cell 4: Bot Configuration (Form UI)
- Generates `config.py` dynamically from interactive form fields.
- Supports all core MLTB features:
  - Custom upload destinations (`DEFAULT_UPLOAD`: `rc`, `gd`, `bh`, `gf`).
  - Google Drive Team Drives, Index URLs, Stop Duplicates.
  - Leech split sizes, document/media mode, dump chats.
  - Sudo users and authorized chats.
  - Built-in torrent search plugins.

### Cell 5: Private Files Helper
- Checks for `token.pickle`, `rclone.conf`, `accounts.zip`, and `.netrc`.
- If found in Google Drive, auto-imports them.
- If not present, offers one-click interactive upload buttons.
- Extracts `accounts.zip` to `accounts/` automatically.

### Cell 6: Start Bot & Real-Time Log Viewer
- Performs pre-flight cleanup of ports `6800`, `8090`, and `8070`.
- Executes `update.py` and `python3 -m bot`.
- Captures stdout and stderr line-by-line in real time.
- Catches `KeyboardInterrupt` (Stop button) to cleanly shut down the bot and background daemons.

### Cell 7: Emergency Stop & Cleanup
- A safety cell to kill any lingering processes:
  ```bash
  pkill -9 -f aria2c; pkill -9 -f qbittorrent-nox; pkill -9 -f sabnzbdplus; pkill -9 -f gunicorn; pkill -9 -f cloudflared; pkill -9 -f "python3 -m bot"
  ```

---

## 🚫 Anti-Seeding Configuration (Zero Upload)

The bot is pre-configured to prevent seeding both during download and after download completion:

1. **During Download:**
   - **qBittorrent:** Global upload slots are set to `0` (`MaxUploads=0`, `MaxActiveUploads=0`, `MaxUploadsPerTorrent=0`) and the global upload rate is throttled to `1 KiB/s` (`GlobalUPSpeedLimit=1`).
   - **Aria2c:** Upload speed limit is capped at the strict minimum (`--max-upload-limit=1K`).
2. **After Download Finishes:**
   - **qBittorrent:** `GlobalMaxRatio=0` and `GlobalMaxSeedingMinutes=0` with `MaxRatioAction=0` (Pause). The torrent is immediately stopped upon reaching 100% and removed once transferred.
   - **Aria2c:** Set to `--seed-ratio=0` and `--seed-time=0`. Aria2 terminates the torrent task the moment the last block is verified.
   - **Bot Logic:** Never pass the `-d` flag in `/mirror` or `/leech`. Without `-d`, the bot automatically pauses and deletes the torrent immediately after uploading.

---

## 🌐 Configuring Cloudflare Tunnel (Torrent Web Selection)

Colab instances are hosted behind Google's private NAT without a public IPv4 address. Usually, this means the torrent file selection web UI (`BASE_URL`) cannot be reached externally.

To solve this:
1. In **Cell 4 (Bot Configuration)**, check `ENABLE_CLOUDFLARE_TUNNEL = True`.
2. The notebook automatically downloads `cloudflared` and launches a quick tunnel pointing to `http://localhost:8088`.
3. It captures the generated `https://xxxx.trycloudflare.com` URL and injects it as `BASE_URL` with `BASE_URL_PORT = 8088`.
4. When you mirror a torrent with the file selection flag in Telegram, clicking the selection link will open the web interface directly in your browser!

---

## 💾 Persisting Credentials on Google Drive

To avoid re-entering your tokens or re-uploading files every time:
1. Ensure `MOUNT_DRIVE = True` in Cell 2.
2. In your Google Drive, navigate to `My Drive` -> `MLTB`.
3. Store any of the following files there:
   - `config.py` (Cell 4 can auto-save this for you by checking `SAVE_CONFIG_TO_DRIVE`)
   - `token.pickle` (Google Drive OAuth)
   - `rclone.conf` (Rclone configuration)
   - `accounts.zip` (Service Accounts JSON archive)
   - `.netrc` (Aria2 direct link credentials)
4. On subsequent Colab runs:
   - Check `LOAD_CONFIG_FROM_DRIVE = True` in Cell 4.
   - Leave `AUTO_IMPORT_FROM_DRIVE = True` in Cell 5.
   - Everything will load automatically in under 5 seconds!

---

## ⏱️ Tips to Prevent Colab Disconnections

1. **Keep the Tab Focused or Active**: Chrome throttles background tabs. Keep the tab open or pinned.
2. **Audio Ping**: Cell 1 initializes an audio context ping to prevent browser-level freezing.
3. **Console Auto-Clicker**: Use the F12 console snippet provided in Cell 1.
4. **Google Colab Limits**:
   - Free Colab runtimes offer up to ~12 hours per session.
   - If disconnected, simply re-run the cells; thanks to Google Drive persistence, re-starting takes less than a minute.

---

## ❓ Troubleshooting & FAQs

### Q: Port 6800 or 8090 is already in use
**A:** Run **Cell 7 (Emergency Stop & Port Cleanup)**. It forcefully terminates any orphan background daemons from a previously stopped run.

### Q: How do I stop the bot?
**A:** Click the **Stop (Interrupt)** button on Cell 6, or run Cell 7.

### Q: Does this support 4GB leech uploads to Telegram?
**A:** Yes! Enter your `USER_SESSION_STRING` in Cell 4 (from a Telegram Premium account) and set `LEECH_SPLIT_SIZE = 4194304000`.

### Q: Where are downloaded files stored?
**A:** Files are temporarily stored in `/app/downloads/` (inside `/content/mirror-leech-telegram-bot/downloads/`). Once a task finishes uploading to Telegram, Google Drive, or Rclone, the bot cleans them up automatically.
