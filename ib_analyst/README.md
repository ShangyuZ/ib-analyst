# IB Analyst — Technical Reference

This folder contains the full project: the Python package, run scripts, example inputs, and sample outputs. Everything needed to install, run, and extend the tool is here.

For a high-level overview, see the [root README](../README.md).

---

## Architecture

| Module | Role |
|---|---|
| `core/ib_analyst/cli.py` | CLI entrypoint — parses flags, runs the pipeline end-to-end |
| `core/ib_analyst/schema.py` | Pydantic model defining the required JSON structure |
| `core/ib_analyst/validators.py` | Financial logic checks — missing FCF, margin anomalies, null fields |
| `core/ib_analyst/client.py` | Thin Anthropic API wrapper |
| `core/ib_analyst/prompt.py` | System prompt and user message templates for Claude |
| `core/ib_analyst/local_analysis.py` | Rule-based analysis engine (no API required) |
| `core/ib_analyst/report_formatter.py` | Renders the note body into Markdown or HTML |
| `core/ib_analyst/sector_benchmarks.py` | Sector comparison data used by the local engine |

---

## Pipeline

```
examples/*.json
      │
      ▼
  schema.py          ← Hard validation — rejects malformed input, exits immediately
      │
      ▼
  validators.py      ← Soft checks — flags missing fields or inconsistencies as warnings
      │
      ├── --use-llm ────► client.py + prompt.py ──► Claude API ──► note body
      │
      └── (default) ────► local_analysis.py ─────────────────────► note body
                                                                        │
                                                                        ▼
                                                              report_formatter.py
                                                                        │
                                                                        ▼
                                                         outputs/{ai,local}/*.{md,html}
```

**Schema validation** (`schema.py`) runs first and is a hard gate — missing required fields cause an immediate exit with a clear error. This prevents silent failures downstream.

**Financial validation** (`validators.py`) runs second and is advisory — it flags things like missing free cash flow or inconsistent margins as warnings but still allows the report to generate. The analyst can decide what to do with incomplete data.

**Analysis** is either AI or local. AI mode sends the structured data and a carefully engineered prompt to Claude; local mode processes the same data through a deterministic rule engine. Both produce the same output shape.

**Formatting** (`report_formatter.py`) wraps the note body in a consistent template with the key metrics header, and renders to Markdown or HTML.

---

## CLI Reference

All flags are passed through the run scripts to `ib-analyst`:

| Flag | Default | Description |
|---|---|---|
| `--input`, `-i` | *(required)* | Path to input JSON file |
| `--output`, `-o` | auto-named | Output file path |
| `--format` | `markdown` | Output format: `markdown` or `html` |
| `--model` | `claude-sonnet-4-6` | Claude model ID (AI mode only) |
| `--use-llm` | off | Enable Claude AI mode |
| `--dry-run` | off | Validate input and print the prompt; skip report generation |

**Common usage:**

```bash
# AI report, default markdown output
./scripts/run_ai.sh

# AI report as HTML with a specific input file
./scripts/run_ai.sh --input examples/AtlasBank_FY2024.json --format html

# Validate input and preview the prompt without generating a report
./scripts/run_ai.sh --dry-run

# Local report with custom output path
./scripts/run_local.sh --output reports/zephyr_note.md
```

The scripts auto-select the most recently modified JSON file from `examples/` when `--input` is not specified.

---

## Design Decisions

**Why is validation split across two layers?**

`schema.py` handles structural integrity — is the JSON well-formed, are required fields present. `validators.py` handles financial logic — is the data internally consistent, are there missing fields that would affect analysis quality. Separating these keeps each concern clean and makes failures easier to diagnose.

**Why does local mode exist?**

Local mode makes the tool usable without an API key or internet access — useful for development, testing new input formats, or validating schema without incurring API cost. The output is lower quality but structurally identical, so it's also a reliable baseline for comparing against AI output.

**Why is the AI optional rather than required?**

The tool is designed to be useful at different stages. During data preparation, local mode gives instant feedback. When producing a final deliverable, AI mode adds the narrative depth. Keeping AI optional makes the tool faster to iterate with and easier to test.

**Why Markdown output?**

Markdown travels well — it renders cleanly on GitHub, pastes into Notion or Confluence without reformatting, and is easy to post-process. HTML is available for cases where a self-contained rendered file is needed, but Markdown is the default.

---

## Adding a New Input

1. Create a JSON file in `examples/` following the structure in `core/ib_analyst/schema.py`
2. Required top-level keys: `company`, `financials`
3. Run `./scripts/run_local.sh --dry-run` to validate before generating a report
4. See `examples/Zephyr_Software_FY2024.json` for a fully populated reference
5. See `examples/AtlasBank_FY2024.json` for a sparse example that tests null handling

---

## Folder Structure

```
ib_analyst/
├── README.md                          ← you are here
├── scripts/
│   ├── run_ai.sh                      ← AI mode entry point
│   └── run_local.sh                   ← local mode entry point
├── examples/
│   ├── Zephyr_Software_FY2024.json    ← tech company, fully populated
│   └── AtlasBank_FY2024.json          ← financial services, sparse data
├── sample_outputs/
│   ├── ai/ZPHY_FY2024_ai_sample.md    ← real AI-generated report
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
