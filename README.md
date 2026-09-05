<div align="center">

# 🖥️ PC Deal Aggregator

**Real-time laptop & PC deal aggregation from Telegram — parsed, scored, categorized, and served over a clean API.**

[![Python](https://img.shields.io/badge/Python-3.10+-3776AB?logo=python&logoColor=white)](https://www.python.org/)
[![FastAPI](https://img.shields.io/badge/FastAPI-0.115-009688?logo=fastapi&logoColor=white)](https://fastapi.tiangolo.com/)
[![Prisma](https://img.shields.io/badge/Prisma-ORM-2D3748?logo=prisma&logoColor=white)](https://www.prisma.io/)
[![PostgreSQL](https://img.shields.io/badge/PostgreSQL-DB-4169E1?logo=postgresql&logoColor=white)](https://www.postgresql.org/)
[![Telethon](https://img.shields.io/badge/Telethon-Telegram-2CA5E0?logo=telegram&logoColor=white)](https://docs.telethon.dev/)
[![spaCy](https://img.shields.io/badge/spaCy-NLP-09A3D5?logo=spacy&logoColor=white)](https://spacy.io/)

</div>

---

## 📖 Overview

**PC Deal Aggregator** listens to PC/laptop resale channels on Telegram, turns each free-form message into a structured deal, and exposes the results through a REST API.

Instead of paying an LLM to read every message, it uses a **hybrid parser** (regex + spaCy NER) that runs locally at zero cost and near-instant speed. Every parsed deal is automatically **categorized** (Gaming, Programming, Budget, Workstation, …) and **scored** based on its specs and price-to-performance ratio, so consumers of the API can rank and filter deals meaningfully.

> **Use case:** power a website, bot, or dashboard that shows the best current laptop deals in a market (originally built around Ethiopian Telegram reseller channels, prices in Birr).

### Why it exists

| | Old approach (AI parser) | This project (hybrid parser) |
|---|---|---|
| **Parsing speed** | 1–3 sec / message | < 0.01 sec / message |
| **Parsing cost** | ~$1–10 per 10k messages | **$0** |
| **Type safety** | ❌ | ✅ (Pydantic + Prisma) |
| **Auto API docs** | ❌ | ✅ (Swagger / ReDoc) |

---

## 🏗️ Architecture

```
                    ┌──────────────────────┐
   Telegram         │   Telethon client    │   real-time listen
   channels  ─────▶ │  (user session)      │   + historical scrape
                    └──────────┬───────────┘
                               │  raw message + image
                               ▼
                    ┌──────────────────────┐
                    │   Hybrid Parser      │   regex  →  structured fields
                    │   (regex + spaCy)    │   spaCy  →  brand / model / title
                    └──────────┬───────────┘
                               ▼
                    ┌──────────────────────┐
                    │  Categorizer + Scorer│   tags + general/category scores
                    └──────────┬───────────┘
                               ▼
                    ┌──────────────────────┐
                    │  PostgreSQL (Prisma) │   deduped by (channel, message)
                    └──────────┬───────────┘
                               ▼
                    ┌──────────────────────┐
                    │   FastAPI  /docs     │   paginated, filterable REST API
                    └──────────────────────┘
```

This repository contains two services under [`pc-deal-aggregator/`](pc-deal-aggregator/):

| Service | Stack | Role |
|---------|-------|------|
| **[`telegram_listner/`](pc-deal-aggregator/telegram_listner/)** | Python · FastAPI · Prisma · PostgreSQL · Telethon · spaCy | **Primary service (v2.0).** Listening, parsing, scoring, storage, and the main REST API. |
| **[`Express-api/`](pc-deal-aggregator/Express-api/)** | Node.js · Express · Mongoose · MongoDB | Legacy/companion read API serving deals from the earlier MongoDB pipeline. |

> The active, maintained service is **`telegram_listner`**. The Express API is kept from the project's first iteration and is optional.

---

## ✨ Features

- 🔌 **Real-time Telegram listener** — reacts to new posts across multiple watched channels.
- 📚 **Historical scraping** — backfill months of past deals from any channel.
- ⚡ **Zero-cost hybrid parser** — regex extracts specs (price, RAM, storage, CPU, GPU, screen, condition, phone numbers, URLs); spaCy NER infers brand/model/title.
- 🏷️ **Automatic categorization** — Gaming, Programming, Graphic Design, Workstation, Student, Budget, Mid-range, Premium, Touchscreen, Ultrabook, 2-in-1, and brand tags.
- 📊 **Deal scoring** — a general score plus per-category scores based on CPU/GPU/RAM/storage tiers and price-to-performance.
- 🗂️ **Rich REST API** — pagination, filtering (price, category, processor, GPU, condition, score), full-text search, and stats.
- 🖼️ **Image handling** — optional download of product images, served as static files.
- 🧾 **Auto-generated docs** — interactive Swagger UI at `/docs` and ReDoc at `/redoc`.
- 🧱 **Type-safe end to end** — Pydantic models + Prisma schema, deduped by `(channelId, messageId)`.

---

## 🛠️ Tech Stack

**Backend:** FastAPI · Uvicorn · Pydantic / pydantic-settings
**Database:** PostgreSQL (e.g. [Neon](https://neon.tech/)) via Prisma ORM (`prisma-client-py`, asyncio)
**Telegram:** Telethon (user session)
**Parsing / NLP:** Python regex + spaCy
**Legacy API:** Node.js · Express · Mongoose · MongoDB

---

## 🚀 Getting Started

### Prerequisites

- **Python 3.10+**
- A **PostgreSQL** database (a free [Neon](https://neon.tech/) instance works well)
- **Telegram API credentials** — create an app at [my.telegram.org](https://my.telegram.org) to get your `API_ID` and `API_HASH`
- *(Optional, for the legacy API)* **Node.js 18+** and a **MongoDB** instance

### 1. Clone

```bash
git clone <your-repo-url>
cd Dream/pc-deal-aggregator/telegram_listner
```

### 2. Install dependencies

```bash
python -m venv .venv
# Windows
.venv\Scripts\activate
# macOS / Linux
source .venv/bin/activate

pip install -r requirements.txt
python -m spacy download en_core_web_sm
```

### 3. Configure environment

Copy the example file and fill in your values:

```bash
cp .env.example .env
```

Key variables (see [`.env.example`](pc-deal-aggregator/telegram_listner/.env.example) for the full list):

```env
# PostgreSQL — pooled URL for the app, direct URL for Prisma migrations
DATABASE_URL=postgresql://user:pass@host-pooler.region.aws.neon.tech/neondb?sslmode=require
DIRECT_DATABASE_URL=postgresql://user:pass@host.region.aws.neon.tech/neondb?sslmode=require

# Telegram — from https://my.telegram.org
TELEGRAM_API_ID=12345
TELEGRAM_API_HASH=your_api_hash
TELEGRAM_PHONE=+2517xxxxxxxx
TELEGRAM_WATCHED_CHANNELS=@pcagregator,@channel2
TELEGRAM_SCRAPE_CHANNELS=@pcagregator,@samcomptech,@sami_brand12

# Server
HOST=0.0.0.0
PORT=8000
DEBUG=True
CORS_ORIGINS=*
```

### 4. Set up the database

The setup script generates the Prisma client and pushes the schema:

```bash
python setup.py
```

Or run the Prisma steps manually:

```bash
prisma generate
prisma db push
```

### 5. Run

```bash
python app/main.py
```

Then open the interactive docs at **http://localhost:8000/docs**.

**First run — Telegram login:** the first time the Telethon client connects it will prompt for the login code sent to your Telegram account and create a `.session` file. You can also log in ahead of time with `python telegram_login.py`.

### 6. Start listening & backfill history

```bash
# Start the real-time listener
curl -X POST http://localhost:8000/api/v1/telegram/start

# Backfill historical deals from a channel
curl -X POST http://localhost:8000/api/v1/telegram/scrape \
  -H "Content-Type: application/json" \
  -d '{"channel": "@pcagregator", "limit": 1000, "days_back": 180}'

# Or run the standalone listener without the API
python start_listener.py
```

---

## 📡 API Reference

Base URL: `http://localhost:8000`

### Health
| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/health` | Basic health check |
| `GET` | `/health/db` | Database connectivity |
| `GET` | `/health/telegram` | Telegram service status |

### Deals
| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/api/v1/deals` | Paginated, filterable list of deals |
| `GET` | `/api/v1/deals/stats` | Aggregate statistics |
| `GET` | `/api/v1/deals/{id}` | A single deal by ID |
| `DELETE` | `/api/v1/deals/{id}` | Delete a deal |
| `GET` | `/api/v1/deals/categories/list` | All known categories |
| `GET` | `/api/v1/deals/brands/list` | All known brands |

**`GET /api/v1/deals` query parameters:** `page`, `page_size`, `sort_by`, `sort_order`, `min_price`, `max_price`, `categories` (comma-separated), `processor`, `graphics_card`, `condition`, `min_score`, `search`.

### Telegram control
| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/api/v1/telegram/start` | Start the live listener |
| `POST` | `/api/v1/telegram/stop` | Stop the live listener |
| `GET` | `/api/v1/telegram/status` | Listener status |
| `POST` | `/api/v1/telegram/scrape` | Backfill history (`{ channel, limit, days_back }`) |

---

## 🗄️ Data Model

Each deal is stored in the `deals` table (see [`prisma/schema.prisma`](pc-deal-aggregator/telegram_listner/prisma/schema.prisma)):

| Field | Notes |
|-------|-------|
| `title`, `model`, `rawMessage`, `imagePath` | Basic info + original message |
| `processor`, `generation`, `ram`, `storage` | Core specs |
| `screenSize`, `resolution`, `graphicsCard`, `graphicsMemory`, `batteryLife`, `condition` | Extended specs |
| `price`, `currency` | Pricing (defaults to `Birr`) |
| `contactNumbers[]`, `urls[]` | Extracted contacts and links |
| `categories[]`, `generalScore`, `categoryScores` (JSON) | Categorization + scoring |
| `channelId`, `messageId`, `postedAt`, `telegramUrl` | Source metadata — unique per `(channelId, messageId)` |

---

## 📂 Project Structure

```
pc-deal-aggregator/
├── telegram_listner/            # Primary Python service (v2.0)
│   ├── app/
│   │   ├── api/                 # FastAPI routers: deals, telegram, health
│   │   ├── core/                # Config (Pydantic) + Prisma client
│   │   ├── models/ entities/    # Pydantic schemas & data classes
│   │   ├── parsers/             # hybrid / regex / advanced / base parsers
│   │   ├── services/            # deal, telegram, categorizer, scorer
│   │   ├── use_cases/           # parse-deal orchestration
│   │   └── main.py              # FastAPI app entry point
│   ├── prisma/schema.prisma     # PostgreSQL schema
│   ├── setup.py                 # One-shot setup (prisma generate + db push)
│   ├── start_listener.py        # Standalone listener
│   ├── scrape_history.py        # Historical scraper
│   ├── requirements.txt
│   └── *.md                     # QUICKSTART, API_DOCUMENTATION, etc.
│
└── Express-api/                 # Legacy Node.js read API (MongoDB)
    ├── app/                     # config, controllers, models, routes, ...
    ├── server.js
    └── package.json
```

Additional docs live inside [`telegram_listner/`](pc-deal-aggregator/telegram_listner/): [QUICKSTART](pc-deal-aggregator/telegram_listner/QUICKSTART.md), [API_DOCUMENTATION](pc-deal-aggregator/telegram_listner/API_DOCUMENTATION.md), [DATABASE_SETUP](pc-deal-aggregator/telegram_listner/DATABASE_SETUP.md), [DEPLOYMENT](pc-deal-aggregator/telegram_listner/DEPLOYMENT.md), and [HYBRID_PARSER_README](pc-deal-aggregator/telegram_listner/HYBRID_PARSER_README.md).

---

## 🧩 Legacy Express API (optional)

The earlier iteration reads deals from MongoDB:

```bash
cd pc-deal-aggregator/Express-api
npm install
# create a .env with your MongoDB connection string
npm run dev        # nodemon server.js  → http://localhost:3000
```

Endpoint: `GET /api/v1/deals`.

---

## 🗺️ Roadmap

- [ ] Authentication (JWT) and rate limiting
- [ ] Amharic / non-Latin text parsing support
- [ ] Price-history tracking & watchlists
- [ ] Email / Telegram deal alerts
- [ ] Analytics dashboard

---

## 🤝 Contributing

The codebase separates concerns cleanly — API in `app/api/`, business logic in `app/services/`, parsing in `app/parsers/`. Contributions are welcome via pull request.

---

<div align="center">

Built with ❤️ using **FastAPI · Prisma · spaCy · Telethon**

</div>
