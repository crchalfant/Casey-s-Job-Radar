# Job Radar

A personal job search automation tool that aggregates listings from 11 sources daily, filters them against your criteria, rates them with Claude AI, and emails you a digest.

## What it does

- Searches 11 sources: ATS direct (Greenhouse/Lever/Ashby), Adzuna, LinkedIn, Brave, Tavily, WeWorkRemotely, Jobicy, Himalayas, Remotive, USAJobs, UltiPro/UKG
- Filters out non-US roles, onsite-outside-your-city roles, below-salary-floor postings, staffing agencies, and category/noise pages
- Rates each job with Claude (Perfect Fit / Good Fit / Worth a Look / Skip) based on your profile
- Emails a daily digest sorted by tier
- Includes a local Kanban dashboard to track your pipeline

## Project structure

```
job-search-profile/
├── scripts/
│   ├── job_radar.py        ← main radar (run this daily)
│   ├── dashboard.py        ← Flask Kanban dashboard
│   └── config.py           ← your personal config (gitignored)
├── config.example.py       ← copy to scripts/config.py and fill in
├── .env.example            ← copy to .env and fill in API keys
├── requirements.txt
└── output/
    └── job-radar/          ← runtime files (gitignored)
```

## Setup

**1. Clone and install dependencies**
```bash
git clone https://github.com/your-username/job-search-profile.git
cd job-search-profile
pip install -r requirements.txt
```

**2. Configure your environment**
```bash
cp .env.example .env
# Edit .env and fill in your API keys
```

**3. Configure your profile**
```bash
cp config.example.py scripts/config.py
# Edit scripts/config.py — add your name, profile, salary floor, target city, and company list
```

**4. Run the radar**
```bash
python scripts/job_radar.py --run
```

Add `--verbose` to see source latencies and filter breakdown.

**5. Run the dashboard**
```bash
python scripts/dashboard.py
# Open http://localhost:5000
```

## Scheduling

To run automatically each morning, add a cron job (macOS/Linux):
```
0 7 * * 1-5 cd /path/to/job-search-profile && python scripts/job_radar.py --run
```

On Windows, use Task Scheduler pointing to `python scripts/job_radar.py --run`.

## API keys needed

| Service | Where to get it | Cost |
|---|---|---|
| Anthropic | console.anthropic.com | Pay per use (~$1-5/month) |
| Brave Search | api.search.brave.com | Free tier: 2000 req/month |
| Tavily | app.tavily.com | Free tier: 1000 req/month |
| Adzuna | developer.adzuna.com | Free |
| USAJobs | developer.usajobs.gov | Free |

Gmail requires an [App Password](https://myaccount.google.com/apppasswords) — not your regular login password.

## Output files

All runtime files live in `output/job-radar/` (gitignored). The radar keeps only the last 3 runs worth of data and prunes automatically.

| File | Purpose |
|---|---|
| `*-jobs.json` | Rated jobs for the dashboard board tab |
| `*-skipped.json` | Skipped jobs for the dashboard skipped tab |
| `*-report.md` | Email report attachment |
| `.seen.json` | Dedup history (60-day rolling window) |
| `board_state.json` | Dashboard card positions and notes |
| `radar_runs.db` | SQLite run history for the Health tab |
| `debug_job_log.txt` | Full job log for debugging source quality |

## Customising

**Adding ATS companies:** Add the company's Greenhouse/Lever/Ashby slug to `COMPANIES` in `config.py`. The radar tries all three APIs automatically.

**Adjusting filters:** Edit the disqualifier lists and salary floor in `config.py` and `job_radar.py`.

**Changing the Claude rating prompt:** Find `RATING_PROMPT` in `job_radar.py` and edit the tier definitions and hard disqualifiers to match your situation.
