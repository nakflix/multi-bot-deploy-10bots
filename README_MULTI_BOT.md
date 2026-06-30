# Deploying 10 Bots (Same Code, Different Token/Port/MongoDB)

This package runs **10 instances of the same bot** using Docker Compose.
Every bot uses the exact same codebase and the exact same settings,
**except** for 4 values per bot:

- `TG_BOT_TOKEN` — the bot's Telegram token (from @BotFather)
- `PORT` — the port the bot's web server listens on
- `DATABASE_URL` — MongoDB connection string
- `DATABASE_NAME` — MongoDB database name

Everything else (APP_ID, API_HASH, CHANNEL_ID, OWNER_ID, force-sub
channels, start message, captions, etc.) is shared across all 10 bots.

## Folder layout

```
multi-bot-deploy/
├── docker-compose.yml      <- defines the 10 bot containers
├── generate_env.sh         <- helper to fill in per-bot values quickly
├── env/
│   ├── bot1.env            <- bot 1's settings
│   ├── bot2.env            <- bot 2's settings
│   ├── ...
│   └── bot10.env           <- bot 10's settings
├── Dockerfile               <- shared build, unchanged from original
├── main.py, levi.py, config.py, plugins/, database/, ...  (original bot code)
```

## Setup (3 steps)

### 1. Fill in each bot's unique values

Open each file in `env/` (bot1.env, bot2.env, ... bot10.env) and replace
the placeholders for that bot:

```
TG_BOT_TOKEN=PUT_BOT1_TOKEN_HERE
PORT=8081
DATABASE_URL=PUT_BOT1_MONGODB_URL_HERE
DATABASE_NAME=PUT_BOT1_DB_NAME_HERE
```

Tip: instead of editing 10 files by hand, you can edit the `BOT_DATA`
array at the top of `generate_env.sh` with all 10 sets of values, then run:

```bash
bash generate_env.sh
```

This regenerates all 10 env files in one go, keeping the shared settings
identical across all of them.

### 2. (Optional) Adjust shared settings

If you want to change something shared by all bots (e.g. CHANNEL_ID,
force-sub channels, captions), edit the "Shared settings" section at the
top of `generate_env.sh` and re-run it — or just edit the shared lines
in each `env/botN.env` file directly.

### 3. Build and start all 10 bots

```bash
docker compose up -d --build
```

This builds one Docker image (shared by all bots) and starts 10
containers from it, each with its own env file.

## Useful commands

```bash
# View logs for a specific bot
docker compose logs -f bot1

# Restart a single bot (e.g. after changing its token)
docker compose restart bot3

# Stop everything
docker compose down

# Stop and start everything
docker compose down && docker compose up -d --build
```

## Notes

- Each bot is mapped to its own host port (8081–8090 by default) so the
  web server inside each container doesn't conflict with the others.
  You can change these in `docker-compose.yml` if those ports are taken
  on your server.
- Each bot should use a **separate Mongo database** (different
  `DATABASE_URL` and/or `DATABASE_NAME`) — otherwise their stored file
  links and settings will collide with each other.
- If you only want to run fewer than 10 bots, just remove the unwanted
  `botN:` blocks from `docker-compose.yml` (or run
  `docker compose up -d --build bot1 bot2 bot3` to start only specific
  ones).
- You can deploy this on any VPS that has Docker and Docker Compose
  installed. Platforms like Heroku, Render, etc. generally only support
  one process per app, so for 10 bots together, a VPS with Docker
  Compose (or 10 separate single-bot deployments on a PaaS) is the
  simplest path.
