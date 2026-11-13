# Telegram Music + Video Bot

## Quick deploy (Linux server)
```bash
sudo apt update && sudo apt install -y python3 python3-venv ffmpeg unzip
unzip musicbot.zip && cd musicbot
cp .env.example .env && nano .env     # set BOT_TOKEN and ADMIN_IDS
bash run.sh                           # creates venv, installs, starts bot
```

## Run 24/7
**systemd:** edit `musicbot.service` (User/paths), then
```bash
sudo cp musicbot.service /etc/systemd/system/
sudo systemctl daemon-reload && sudo systemctl enable --now musicbot
sudo journalctl -u musicbot -f        # logs
```
**Docker:**
```bash
docker build -t musicbot .
docker run -d --restart always --env-file .env -v musicbot_data:/data musicbot
```
**Quick:** `nohup bash run.sh > bot.log 2>&1 &`  or  `screen -S bot` then `bash run.sh`

## Setup checklist
1. Create the bot in @BotFather, put the token in `.env` (never share it).
2. Send `/id` to the bot, put your ID in `ADMIN_IDS`, restart.
3. `/admin` -> add force-join channels/groups (bot must be admin there).
4. `/groups` -> manage groups, settings, stats, broadcast.
5. For plain-text search in groups: make the bot a group admin or disable privacy in BotFather.
6. If YouTube blocks the server, export `cookies.txt` and set `COOKIES_FILE=cookies.txt`.

## Admin commands
/admin /groups [search] /ban [global] /unban [global] /warn /approve <id> /unapprove <id>

Data is stored in `musicbot_data.json` (back it up).
Bot Clone: `/clones` `/cloneunblock <id>` (see "v4 — Bot Clone" below).

## Commands (updated)

- `/song <name> a` → audio · `/song <name> v` → video (always 480p) · `/song <name>` → pick from a list
- `/search <name>` → numbered results list (title, channel, duration, views) — tap a number to pick
- `/lyrics <song>` — lyrics (also a 🎤 button on every song card)
- `/lang` — English / اردو / हिन्दी
- `/stats` — bot totals + your own downloads (admins see extra info)
- Inline mode: type `@yourbot song name` in any chat (enable it in @BotFather: `/setinline`)
- Wrong command (e.g. `/sog`) → the bot suggests the right one
- Queue: when the server is busy users see "You're #N in line"
- Admin alerts: admins get a Telegram message when downloads keep failing (e.g. cookies expired) and when it recovers

Every bot message ends with: *Developed by Dock — @Dockworldwide*

## Force-join
1. Add the bot as **admin** in your channel/group.
2. In private chat send `/admin` → **➕ Add channel / group** → send `@username`, the chat ID, or forward a post from it.
3. Users must join before they can use any command, text search, button or inline query. They tap **✅ I've Joined** to verify.
4. If the bot loses admin rights in a required chat, admins get an alert.

Buttons use Telegram's coloured styles (blue / green / red). Requires `python-telegram-bot>=22.7`: `pip install -U -r requirements.txt`.

## Song recognition
Send a voice note / audio / video clip to the bot in private chat (or reply to one with `/find`, also in groups) and it identifies the song, then shows the Audio / Video 480p / Lyrics card.
Set `AUDD_TOKEN` in `.env` (free token: https://dashboard.audd.io). Default limit: 5 recognitions per user per day (`RECOGNIZE_DAILY_LIMIT`).

## Speed
- Searches and info lookups run in their own thread pool, so they never wait behind downloads.
- Repeated searches are answered instantly from cache (30 min); identical simultaneous searches share one request.
- Tune parallel downloads with `MAX_PARALLEL_DOWNLOADS` in `.env` (default 6; lower it on a small server).

## What's new in v3

**Artist & song info**
- `/artist <name>` — artist card (type, country, active since, genres), latest releases, top songs as tappable numbers, and a 🔔 Follow button.
- **ℹ️ Info** button on every song card — album, release year, genre (MusicBrainz), plus BPM and key.
  MusicBrainz has **no BPM/key data**. BPM now works **with no key at all** (free Deezer lookup, used automatically; Deezer has no key/scale). For musical key too, use `GETSONGBPM_KEY` (free key, recommended) or, if you `pip install librosa`, an **🎚 Analyze BPM & key** button that estimates them from the audio.

**Share card** — **🔗 Share** on any song sends a card with `t.me/<bot>?start=song_<id>`; opening it shows the song card directly.

**Release alerts** — `/follow <artist>`, `/following`. Checked every `RELEASE_CHECK_HOURS` (default 6). Free: 5 artists, premium: 25.

**Referrals** — `/invite` gives a personal link. Each friend who starts the bot through it adds `REF_BONUS` daily recognitions (cap `REF_MAX_BONUS`). The flat `RECOGNIZE_DAILY_LIMIT` is now the base value.

**Premium (JazzCash / Easypaisa, manual)**
1. `/admin` → **⭐ Premium & Sponsor** → set price, duration, JazzCash and Easypaisa number + account name.
2. Users open `/premium`, pick a method, pay, tap **✅ I've Paid** and send the transaction ID or a screenshot.
3. Every admin gets the proof with **Approve / Reject** buttons. Approve activates premium and messages the user.
4. Give premium directly by chat ID: `/givepremium <chat_id> [days]` (or ➕ in the panel). Remove: `/removepremium <chat_id>`.

Premium perks: **priority queue** (skips ahead of free users in `queued_download`), `PREMIUM_RECOG_LIMIT` recognitions/day, no sponsor line, `/brand`, more followed artists.

**Sponsor slot** — admin panel → *Set sponsor line* / ON-OFF / Clear. Shown (in addition to the credit) on search results, song cards, the start screen and delivered files, never to premium users in private chat.

**Group footer** — in a group, a group admin with premium runs `/brand Your text` (`/brand off` to reset). It replaces "Developed by Dock" only in that group, and only while the setter's premium is active (set `BRANDING_FREE=1` to let every group admin do it).

New data (premium, pending payments, referrals, followed artists, sponsor, pay settings) is stored in `musicbot_data.json` — back it up.

BPM data provided by [GetSongBPM](https://getsongbpm.com)


## v4 — Bot Clone

Users can run their own copy of this bot (own token, own users/settings/data) from a **🤖 Bot Clone** button next to the other `/start` buttons.

**User flow**
1. Tap **🤖 Bot Clone** → read and accept the agreement (*"…you will not revoke or change your bot token or delete your bot after cloning. If you violate these rules, your access to the Bot Clone feature will be permanently blocked."*).
2. Send the **bot token** → verified live with Telegram (`getMe`); the message with the token is deleted right away.
3. Send their **Chat ID** (or tap *Use my ID*) → they become an admin of **their clone only**.
4. Send a **profile picture** → set automatically on the clone (`setMyProfilePhoto`).
5. Tap **🚀 Submit Request**.

**Admin flow** — every main admin receives the request (user, bot, token in a spoiler, admin chat ID, agreement time, photo) with **✅ Approve & Clone / ❌ Reject**. Nothing starts before approval; on approval the clone starts automatically and the user is told how to open it.

**Admin access** — each clone runs as its own process with `ADMIN_IDS = main admin(s) + the clone owner`. Both can use `/admin`, `/groups`, `/ban`, … on that clone; the owner has **no** rights on the main bot.

**Rule enforcement** — the main bot re-checks every clone token every `CLONE_CHECK_MINUTES` (default 10). If Telegram rejects the token twice in a row (revoked/changed token or deleted bot) the clone is stopped and the owner is **permanently blocked** from Bot Clone (admins are notified and can unblock).

**Main-admin commands** (main bot only): `/cloneon` / `/cloneoff` (feature switch — hides the button and blocks new requests; running clones keep running), `/clones` (list, on/off switch, stop, start, remove, unblock; also under *Admin Panel → 🤖 Bot Clones*), `/cloneunblock <user_id>`.

**Settings (.env, optional)**: `CLONE_MAX_PER_USER` (default 1), `CLONE_CHECK_MINUTES` (default 10).

**Notes**
- Clone data lives in `clones/<bot_id>/` next to `DATA_FILE` (so on Railway it's inside the `/data` volume — keep the volume). Clone tokens are stored there in plain text; protect the volume/backups.
- Each clone is one extra Python process (~100–150 MB RAM); size your server accordingly.
- A bot token can only be polled from one place: tell users to stop their bot elsewhere before cloning.
- **Inline troubleshooting:** admins can run `/inlinecheck` in any bot (main or clone) — it asks Telegram whether inline mode is ON and gives the fix if not. Inline answers are capped at ~6.5 s (Telegram drops slower ones) and always show a visible result (hint / still searching / no results), so "nothing appears" means inline mode is OFF in @BotFather. Unhandled errors are now sent to the admins.
- **Inline mode** (`@botname song name` in any chat) must be enabled by each bot's owner in @BotFather → `/setinline`; Telegram doesn't allow bots to switch it on. After cloning, the owner is told this and gets a **🔄 Check inline mode** button (uses `getMe → supports_inline_queries`).
- Clones restart automatically if they crash, and are brought back up when the main bot restarts.

## New-user notifications
When someone sends `/start` to a bot for the first time, that bot's admin gets a **🆕 New User Joined!** message (name, @username, Chat ID, bot type, status). Once per user per bot.
- **Main bot** → the main admin(s) (`ADMIN_IDS`) only.
- **Cloned bot** → only that clone's owner (`CLONE_OWNER_ID`, set automatically) — never the main admin.
- Every bot keeps its own list (`welcomed` in its own data file). Users that already existed when this feature was installed are not announced.
- Admins starting their own bot aren't announced. An admin must have opened the bot (pressed Start) to receive messages from it.
