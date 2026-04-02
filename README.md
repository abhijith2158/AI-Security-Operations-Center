# Live AI Security Operations Center

A local, laptop-scale Security Operations Center (SOC) demo built with Streamlit, rule-based heuristics, and a local Ollama model.

This project watches security logs, groups suspicious activity into higher-signal batches, asks a local LLM for triage, and records confirmed threats in a CSV audit trail. It is designed to be practical for demos, student projects, and portfolio reviews: the architecture is small enough to understand quickly, but still shows real security workflow ideas.

## Why this project is interesting

- Uses a hybrid detection pipeline instead of calling an LLM on every line.
- Separates live monitoring from shared detection logic.
- Supports local-first AI with Ollama instead of a cloud dependency.
- Keeps an audit history of confirmed threats for reporting and analytics.
- Includes optional Telegram alerting for real-time notification.

## Repository layout

```text
.
|-- app.py
|-- live_monitor.py
|-- soc_engine.py
|-- requirements.txt
|-- sample_data/
|   |-- security.sample.log
|   `-- alerts_history.sample.csv
`-- .streamlit/
    `-- secrets.example.toml
```

## What each file does

- `app.py`: offline log-upload dashboard for scanning saved `.log` and `.txt` files.
- `live_monitor.py`: real-time dashboard that tails `security.log`, batches events, and sends Telegram alerts for confirmed threats.
- `soc_engine.py`: shared SOC engine with parsing, scoring, batching, Ollama prompting, verdict parsing, CSV export, and log tailing.
- `sample_data/security.sample.log`: safe example input for demos and screenshots.
- `sample_data/alerts_history.sample.csv`: safe example output for reporting demos.

## Detection design

The project uses a two-stage approach to reduce false positives and unnecessary LLM calls.

### 1. Heuristic scoring

Each log line is assigned a suspicion score using simple, deterministic indicators such as:

- failed authentication language
- unauthorized or denied access
- exploit-related keywords such as `sql injection`, `xss`, `rce`, and `webshell`
- presence of an IP address

Only lines above a configurable threshold are buffered for model analysis.

### 2. Batched AI triage

Suspicious events are grouped by IP so the model can judge behavior patterns instead of isolated lines. This makes the system better at distinguishing:

- `HUMAN_ERROR`: a few irregular mistakes
- `SUSPICIOUS`: unclear activity worth monitoring
- `CONFIRMED_THREAT`: repeated, patterned, automation-like behavior

For failed-login bursts, the engine also considers event timing inside a sliding window so that fast, regular attempts are treated differently from occasional human typos.

## Features

- Real-time log tailing with offset-based deduplication
- Shared engine for both live and offline workflows
- Configurable heuristic and batching thresholds in the UI
- Local Ollama integration over HTTP
- Telegram alerts for confirmed threats
- CSV audit trail for analytics and reports
- Optional Windows firewall block action from the UI

## Requirements

- Python 3.10 or newer
- [Ollama](https://ollama.com/) installed locally
- A local model pulled into Ollama, such as `llama3.2`
- Windows if you want to use the built-in firewall block action

## Installation

### 1. Create a virtual environment

```powershell
python -m venv venv
.\venv\Scripts\activate
```

### 2. Install Python dependencies

```powershell
pip install -r requirements.txt
```

### 3. Start Ollama and pull a model

```powershell
ollama pull llama3.2
```

Make sure the Ollama service is running before you start the Streamlit apps.

## Configuration

You can configure secrets in either of these locations:

- `.streamlit/secrets.toml`
- `.env`

Safe starter files are included:

- `.streamlit/secrets.example.toml`
- `.env.example`

Required Telegram settings for live alerting:

```toml
TELEGRAM_TOKEN = "replace-with-your-bot-token"
TELEGRAM_CHAT_ID = "replace-with-your-chat-id"
```

The live dashboard requires Telegram credentials. The offline dashboard can run without them.

## Running the apps

### Live SOC dashboard

```powershell
streamlit run live_monitor.py
```

Expected behavior:

- watches `security.log` in the project root
- batches suspicious events by IP
- sends Telegram alerts only for `CONFIRMED_THREAT`
- appends confirmed detections to `alerts_history.csv`

### Offline scan dashboard

```powershell
streamlit run app.py
```

Expected behavior:

- accepts uploaded `.log` and `.txt` files
- analyzes suspicious lines with the same engine used by the live dashboard
- appends confirmed detections to `alerts_history.csv`
- displays simple analytics from the CSV history

## Demo data

If you want a clean GitHub demo without exposing local runtime data, use the files in `sample_data/`:

- `sample_data/security.sample.log`
- `sample_data/alerts_history.sample.csv`

These are safe examples intended for screenshots, walkthroughs, and reviewer demos.

## Public repository safety

This repository is set up to avoid committing sensitive or machine-specific files:

- `venv/` is ignored
- `__pycache__/` is ignored
- `.env` is ignored
- `.streamlit/secrets.toml` is ignored
- `security.log` is ignored
- `alerts_history.csv` is ignored

That means you can keep real local credentials and runtime logs on your machine without publishing them.

## Suggested GitHub description

`Local AI-powered SOC demo built with Streamlit, Ollama, heuristic triage, and batched threat classification.`

## Future improvements

- Add unit tests for parsing and scoring logic
- Add a Docker setup for easier demo environments
- Add Linux/macOS response actions alongside Windows firewall support
- Add richer charts and historical filtering
- Add structured JSON log ingestion

## Disclaimer

This project is a demo and learning tool, not a production-grade SOC platform. LLM output can be imperfect, so confirmed incidents should still be reviewed by a human analyst in real-world deployments.
