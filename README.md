# Trigger-Based Sales Intelligence Engine (TBSIE)

A decentralized autonomous pipeline that transforms real-world business events into hyper-personalized sales outreach in under 10 minutes.

## Architecture

```
  Workflow 1: The Tactical Event Listener

  RSS Feeds --> n8n --> Groq LLM (NER + Scoring)
                          |
                          v
                 FastAPI + Playwright Scouter
                 (Visual Web Recon)
                              |
                              v
  Workflow 2: The Strategic Closer

  SQLite (Unposted Leads) --> n8n Polling --> AI Copy
                          |
                          v
              Slack Block Kit (HITL Approval)
                          |
                          v
                 Resend API (Email Delivery)
                 SQLite State Sync (Mark Posted)
```

## Workflow 1: The Tactical Event Listener

**Stack:** n8n, Groq (Llama 3.3), RSS, Python (FastAPI + Playwright)

1. **Ingestion** — Scans global RSS feeds for trigger events: funding rounds, leadership changes, M&A.
2. **Intelligence** — Groq LLM performs Named Entity Recognition (NER), extracts company names, assigns Lead Score (1–10).
3. **Refinery** — Python/Pyodide node flattens AI output into integers, high-pass filters at score 7+.
4. **Action** — Triggers Playwright Scouter via API for visual web recon and URL extraction.

## Workflow 2: The Strategic Closer

**Stack:** n8n, FastAPI, SQLAlchemy, Slack Block Kit, Resend API

1. **State Management** — Pull-based polling retrieves "unposted" leads from SQLite.
2. **Synthesis** — AI copywriter drafts 2-sentence pitch referencing the trigger event.
3. **Human-in-the-Loop** — Slack notification with lead card + self-generating resume link. Pauses until human clicks approve.
4. **Safe Execution** — On approval: sends email via Resend, marks posted in SQLite. Idempotency guaranteed.

## Tech Stack

| Layer | Technology |
|-------|------------|
| Orchestration | n8n (workflow automation) |
| AI/NER | Groq (Llama 3.3 70B) |
| Web Recon | Python, FastAPI, Playwright |
| Storage | SQLite, SQLAlchemy |
| Notifications | Slack Block Kit |
| Email | Resend API |
| Hosting | Docker, Cloudflare Tunnel |

## Local Deployment (n8n)

n8n runs locally on Docker (no 14-day cloud trial, no usage limits, full data privacy). Cloudflare Tunnel exposes it online so external apps (Slack webhooks, Resend callbacks) can reach it:

```bash
# Run n8n locally
docker run -d --restart always \
  --name n8n \
  -p 5678:5678 \
  -v n8n_data:/home/node/.n8n \
  -e WEBHOOK_URL=https://n8n.yourdomain.com \
  n8nio/n8n

# Expose via Cloudflare Tunnel (free)
cloudflared tunnel create n8n
cloudflared tunnel route dns n8n n8n.yourdomain.com
cloudflared tunnel run n8n
```

Limitation: Cloudflare free plan caps payloads at 50MB and doesn't proxy WebSocket — sufficient for HTTP webhook workflows.

## Setup

```bash
git clone https://github.com/MUIZ-UDDIN/tbsie.git
cd tbsie

# Backend
pip install fastapi uvicorn playwright sqlalchemy
playwright install
uvicorn main:app --reload

# Environment variables
GROQ_API_KEY=gsk_your_key
RESEND_API_KEY=re_your_key
SLACK_WEBHOOK_URL=https://hooks.slack.com/...

# Import n8n workflows from /n8n-workflows
```

## Environment Variables

```
GROQ_API_KEY=           # Groq API key
RESEND_API_KEY=         # Resend API key
SLACK_WEBHOOK_URL=      # Slack webhook for HITL
DATABASE_URL=sqlite:///leads.db
SCOUTER_API_URL=http://localhost:8000/scout
```

## Screenshots

| Screenshot | Description |
|------------|-------------|
| <img width="3170" height="1361" alt="image" src="https://github.com/user-attachments/assets/a1da88da-cb40-4a4a-9bef-ab06291a73e4" /> | n8n workflow: Polling -> AI copy -> Slack HITL -> Resend |
| <img width="3080" height="1277" alt="image" src="https://github.com/user-attachments/assets/5ab8793d-1e24-4172-b5d7-47b2af52755b" /> | n8n workflow: RSS -> LLM scoring -> Scouter API |

## Blog Post

Full walkthrough: [How to Build a Trigger-Based Sales Intelligence Engine](https://muizuddin.com/blog/trigger-based-sales-intelligence-engine)

Full walkthrough: [How to Self-Host n8n for Free Without the 14-Day Trial (Docker + Cloudflare Tunnel Guide)](https://muizuddin.com/blog/self-host-n8n-free-docker-cloudflare-tunnel)

## License

MIT
