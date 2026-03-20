# IB Analyst — Technical Reference

## Architecture

| File | Role |
|------|------|
| `core/ib_analyst/cli.py` | Typer CLI entrypoint — parses flags, orchestrates the pipeline |
| `core/ib_analyst/schema.py` | Pydantic model that defines the expected JSON structure |
| `core/ib_analyst/validators.py` | Financial logic checks (warnings, not hard errors) |
| `core/ib_analyst/client.py` | Thin wrapper around the Anthropic API |
| `core/ib_analyst/prompt.py` | System prompt and user message templates for Claude |
| `core/ib_analyst/local_analysis.py` | Rule-based analysis engine (no API required) |
| `core/ib_analyst/report_formatter.py` | Renders note body into Markdown or HTML |
| `core/ib_analyst/sector_benchmarks.py` | Sector comparison data used by local analysis |

---

## Pipeline

```
examples/*.json
      │
      ▼
  schema.py          ← Pydantic validation (hard fail on bad structure)
      │
      ▼
  validators.py      ← Financial logic checks (warnings only)
      │
      ├── AI mode ──────► client.py + prompt.py ──► Claude API ──► note body
      │
      └── Local mode ───► local_analysis.py ──────────────────────► note body
                                                                         │
                                                                         ▼
                                                               report_formatter.py
                                                                         │
                                                                         ▼
                                                              outputs/{ai,local}/*.{md,html}
```

---

## CLI Reference

All flags are passed through the run scripts to `ib-analyst` (defined in `cli.py`):

| Flag | Default | Description |
|------|---------|-------------|
| `--input`, `-i` | *(required)* | Path to input JSON file |
| `--output`, `-o` | auto-named | Output file path |
| `--format` | `markdown` | Output format: `markdown` or `html` |
| `--model` | `claude-sonnet-4-6` | Claude model (AI mode only) |
| `--use-llm` | off | Enable Claude AI mode |
| `--dry-run` | off | Validate input and print prompt; skip report generation |

Examples:

```bash
./scripts/run_ai.sh --format html
./scripts/run_ai.sh --input examples/AtlasBank_FY2024.json --format html
./scripts/run_local.sh --output my_note.md
./scripts/run_ai.sh --dry-run
```

---

## AI Mode vs Local Mode

| | Local mode | AI mode |
|--|------------|---------|
| API key required | No | Yes (`ANTHROPIC_API_KEY`) |
| Output quality | Rule-based template | Full analyst narrative |
| Speed | Instant | ~10–20 seconds |
| Use case | Dev/testing | Final reports |

---

## Adding a New Example Input

1. Create a JSON file in `examples/` following the schema in `core/ib_analyst/schema.py`
2. Required top-level keys: `company`, `financials`
3. Run `./scripts/run_local.sh --dry-run` to validate before generating a report
4. See `examples/Zephyr_Software_FY2024.json` for a fully populated reference

---

## Folder Structure

```
ib_analyst/
├── README.md                        ← you are here
├── scripts/
│   ├── run_ai.sh                    ← main entry point (Claude AI)
│   └── run_local.sh                 ← local rule-based mode
├── examples/
│   ├── Zephyr_Software_FY2024.json  ← fully populated (tech company)
│   └── AtlasBank_FY2024.json        ← sparse data (financial services)
├── sample_outputs/
│   ├── ai/ZPHY_FY2024_ai_sample.md
│   └── local/ZPHY_FY2024_local_sample.md
├── core/
│   └── ib_analyst/
│       ├── cli.py
│       ├── client.py
│       ├── prompt.py
│       ├── local_analysis.py
│       ├── report_formatter.py
│       ├── schema.py
│       ├── sector_benchmarks.py
│       └── validators.py
├── pyproject.toml
├── .env.example
└── .gitignore
```
