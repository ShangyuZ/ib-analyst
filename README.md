# IB Analyst Note Generator

A CLI tool that turns a structured JSON file of company financials into a professional investment banking analyst note — via Claude AI or a local rule-based engine.

**The problem:** Writing IB analyst notes from raw financial data is time-consuming and format-intensive. This tool automates the full pipeline: validate the input, run analysis, and render a clean Markdown or HTML report — in seconds.

---

## Prerequisites

- Python 3.10+
- Anthropic API key (only required for AI mode)

---

## Setup

```bash
cd ib_analyst
pip install -e .
cp .env.example .env
# Edit .env and set: ANTHROPIC_API_KEY=sk-ant-...
```

---

## Run it

```bash
cd ib_analyst
./scripts/run_ai.sh      # Claude AI narrative (requires API key)
./scripts/run_local.sh   # Rule-based output (no API key needed)
```

---

## More

- [Technical docs →](ib_analyst/README.md) — architecture, CLI reference, pipeline walkthrough
- [Sample outputs →](ib_analyst/sample_outputs/) — see what the tool produces before running it
