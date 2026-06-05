# AI News Aggregator 

Here’s how to run this project end to end.

1. Prerequisites
Python 3.12+
uv (or use the existing .venv)
Docker (for PostgreSQL)
OpenAI API key (for digests / email ranking)
Gmail app password (only if you want the email step to work)
2. One-time setup
From the project root:

cd /Users/rakshitanalwaya/Desktop/ai-news-aggregator-master
Install dependencies:

uv sync
# or: source .venv/bin/activate
Environment variables — copy the template and fill in real values:

cp app/example.env .env
Edit .env (not example.env):

Variable	Purpose
OPENAI_API_KEY
AI digests + email curation
MY_EMAIL
Your Gmail address
APP_PASSWORD
16-char Gmail app password (not normal password)
POSTGRES_*
Database (defaults usually fine)
Start PostgreSQL:

docker compose -f docker/docker-compose.yml up -d
Create database tables:

.venv/bin/python -m app.database.create_tables
3. Run the full pipeline (recommended)
This runs all 5 steps: scrape → process → digest → email.

.venv/bin/python main.py
Optional args: hours and top_n (articles in email):

.venv/bin/python main.py 48 10   # last 48 hours, top 10 articles
Same thing via the daily runner:

.venv/bin/python -m app.daily_runner
4. Run individual steps (for debugging)
Always run from the project root using -m:

Step	Command
Scrape OpenAI RSS
.venv/bin/python app/scrapers/openai.py
Scrape Anthropic
.venv/bin/python app/scrapers/anthropic.py
Generate digests
.venv/bin/python -m app.services.process_digest
Send email digest
.venv/bin/python -m app.services.process_email
Test email only
.venv/bin/python app/services/email_delivery.py
5. What “success” looks like
The pipeline logs progress like:

Scraping (YouTube, OpenAI, Anthropic)
Processing Anthropic markdown
YouTube transcripts
Creating digests (OpenAI API — watch for 429 rate limits)
Sending email digest
If step 5 fails, earlier steps may still have worked; data will be in Postgres.

**Common issues**
ModuleNotFoundError: No module named 'app' — run from repo root with python -m ..., not python app/services/foo.py (unless the file has sys.path fixes).
OPENAI_API_KEY missing — ensure .env is in the project root and you run from there.
Gmail auth errors — use a 16-character app password, not your normal Gmail password.
DB connection errors — confirm Docker Postgres is up: docker compose -f docker/docker-compose.yml ps


**Quick start (copy-paste):**

cd /Users/rakshitanalwaya/Desktop/ai-news-aggregator-master
uv sync
cp app/example.env .env          # then edit .env with your keys
docker compose -f docker/docker-compose.yml up -d
.venv/bin/python -m app.database.create_tables
.venv/bin/python main.py
