# Job Agent — Autonomous Job Application Pipeline

A multi-agent Python system that autonomously searches job boards, scores listings with AI, filters by location, and delivers a daily digest email — built as a personal job search tool while actively job hunting.

**[View Live Demo](https://sdellin12.github.io/job-agent-demo)**

---

## What It Does

The pipeline runs hourly (7am–6pm via Windows Task Scheduler) and handles the entire job discovery workflow without manual intervention:

1. Scrapes 6+ job boards simultaneously using RSS feeds and REST APIs
2. Deduplicates listings across sources and filters against a 30-day seen-jobs cache
3. Runs each listing through a Haversine distance check (remote or within 50mi of home)
4. Scores every qualifying listing 1-10 using Claude AI against a detailed candidate profile
5. Sends a single HTML digest email with all results via Gmail API
6. Logs everything to Google Sheets for historical tracking

A local Flask dashboard provides full visibility — status tracking, search/filter, and on-demand AI-generated resumes and cover letters.

---

## Architecture

```
orchestrator.py          — pipeline coordinator
agents/
  job_scout.py           — scraping, dedup, location filter, AI scoring
  dispatcher.py          — digest email + Google Sheets logging
  resume_tailor.py       — on-demand resume generation (Claude API)
  cover_letter.py        — on-demand cover letter generation (Claude API)
dashboard.py             — Flask web dashboard
utils/
  location_filter.py     — Haversine distance + remote detection
templates/
  dashboard.html         — CRT terminal UI
resume/
  master_resume.json     — candidate profile used by all agents
```

---

## Key Features

**Multi-source job scraping**
- Indeed RSS (30+ targeted searches across keywords and locations)
- USAJobs federal API
- Adzuna API
- RemoteOK, Remotive, Jobicy, We Work Remotely
- Rotating keyword bank — each run uses a different search term batch so coverage expands over time

**Smart deduplication and caching**
- URLs deduplicated across all sources within each run
- Seen-jobs cache stored as `{url: date}` — entries expire after 30 days so job listings automatically re-enter the pool without manual intervention

**Haversine location filtering**
- Two-pass filter: remote detection first (title, location field, description), then coordinate lookup
- 60+ city coordinate database covering WV, VA, MD, DC, PA
- Full state names normalized to abbreviations before lookup (`"Harrisonburg, West Virginia"` → `"harrisonburg wv"`)
- Hard-reject list for clearly out-of-region states (FL, CA, TX, WA, etc.)
- Non-tech job titles (nurse, attorney, pharmacist, etc.) rejected before location check to save API tokens

**AI scoring with cost optimization**
- Claude Haiku model (cheapest available, $1/$5 per million tokens)
- Static system prompt keeps per-job token count under 200 input tokens
- Scoring prompt trimmed to avoid sending unnecessary context
- Typical full run cost: under $0.05

**Flask dashboard**
- CRT terminal aesthetic (green phosphor on black, scanline overlay, Orbitron + Share Tech Mono fonts)
- Job status lifecycle: Found → Applied → Interview → Offer / Rejected
- Filter by status, search by title or company
- On-demand resume and cover letter generation — Claude only called when you click Generate, not on every pipeline run
- Live pipeline log with color-coded output
- PDF served from local output directory

---

## Tech Stack

| Layer | Tools |
|---|---|
| Language | Python 3.12 |
| AI | Anthropic Claude API (Haiku model) |
| Web dashboard | Flask, HTML/CSS/JS |
| Email | Gmail API (OAuth2) |
| Logging | Google Sheets API |
| Job sources | feedparser, requests (Adzuna, USAJobs, RemoteOK APIs) |
| PDF generation | ReportLab |
| Scheduling | Windows Task Scheduler |
| Location math | Haversine formula (no external geocoding API) |

---

## Setup

### Prerequisites

- Python 3.12+
- Anthropic API key
- Google Cloud project with Gmail API and Sheets API enabled
- USAJobs API key (free at developer.usajobs.gov)
- Adzuna API credentials (free at developer.adzuna.com)

### Installation

```bash
git clone https://github.com/sdellin12/job-agent
cd job-agent
python -m venv .venv
.venv\Scripts\activate       # Windows
pip install -r requirements.txt
```

### Configuration

Copy `.env.example` to `.env` and fill in your credentials:

```env
ANTHROPIC_API_KEY=your_key_here
USAJOBS_API_KEY=your_key_here
USAJOBS_EMAIL=your_email@example.com
ADZUNA_APP_ID=your_id_here
ADZUNA_APP_KEY=your_key_here
RECIPIENT_EMAIL=your_email@example.com
```

### Google OAuth setup

```bash
python setup_google_auth.py
```

Follow the browser prompt to authorize Gmail and Sheets access. Credentials are saved locally and not committed.

### Configure your location

In `utils/location_filter.py`, update:

```python
HOME_COORDS = (38.8731, -78.8665)   # your lat/lon
HOME_LABEL  = "Your City, ST"
MAX_MILES   = 50
```

### Run manually

```bash
python orchestrator.py --min-score=5
python orchestrator.py --min-score=5 --dry-run   # no email sent
```

### Run the dashboard

```bash
python dashboard.py
# open http://localhost:5000
```

### Schedule with Task Scheduler (Windows)

Create `run_pipeline.bat`:

```batch
@echo off
cd /d C:\path\to\job-agent
call .venv\Scripts\activate
python orchestrator.py --min-score=5
```

In Task Scheduler: Daily trigger, start 7:00 AM, repeat every 1 hour for 11 hours.

---

## Customization

**Scoring criteria** — edit the `SCORING_SYSTEM` prompt in `agents/job_scout.py` to match your skills and target roles.

**Candidate profile** — edit `resume/master_resume.json`. This file is used by all agents for scoring context and document generation.

**Search keywords** — the `ROTATING_KEYWORDS` list in `job_scout.py` cycles through keyword batches across runs. Add your own target role terms here.

**Score threshold** — `--min-score=5` controls the cutoff. Lower = more results, higher = stricter filtering.

---

## Cost

Using Claude Haiku with optimized prompts, a typical full run (200-300 jobs fetched, 10-20 reaching the scoring stage after filters) costs **$0.02 to $0.05**. Running hourly for a full day costs roughly **$0.30 to $0.50**.

---

## Project Background

Built this while actively searching for work after a layoff. The goal was to solve a real problem — manually checking job boards every day is tedious, but applying to everything is wasteful. The agent handles discovery and initial filtering so I only spend time on genuinely relevant opportunities.

The scoring system, location filter, and dashboard all evolved through real use and real feedback from actual pipeline runs.

---

## Notes

- Personal data (email, location, resume content) has been removed from this public repo
- See `.env.example` for required environment variables
- The demo at the link above uses sanitized sample data with fictional company names
- Company names in actual runs are never shared publicly

---

## License

MIT — use freely, attribution appreciated.
