---
name: memory-pipeline
description: AI agent memory consolidation system that extracts structured facts from conversations and daily notes, builds a knowledge graph with semantic links, and generates daily briefings. Use when working on memory management, briefing generation, knowledge consolidation, or improving agent context retention across sessions.
---

# Memory Pipeline

A complete memory consolidation system for Clawdbot AI agents that transforms raw conversation history and daily notes into structured knowledge.

## What This Does

The memory-pipeline is a three-stage system that helps AI agents maintain long-term memory:

1. **Extract** — Pulls structured facts (decisions, preferences, learnings, commitments) from daily notes and session transcripts using LLM extraction
2. **Link** — Builds a knowledge graph with embeddings and bidirectional links between related facts, identifies contradictions
3. **Briefing** — Generates a compact BRIEFING.md file loaded at session start with personality reminders, active projects, recent decisions, and key context

## Quick Start

### Installation

The skill includes three Python scripts in `scripts/`:
- `memory-extract.py` — Fact extraction
- `memory-link.py` — Knowledge graph building
- `memory-briefing.py` — Daily briefing generation

All scripts auto-detect your workspace from:
1. `CLAWDBOT_WORKSPACE` environment variable
2. Current working directory (if contains SOUL.md or AGENTS.md)
3. `~/.clawdbot/workspace` (default fallback)

### Requirements

**At least one LLM API key** is required:
- OpenAI API key (for GPT-4o-mini + embeddings)
- Anthropic API key (for Claude Haiku)
- Gemini API key (for Gemini Flash)

Set via environment variable or config file:
```bash
# Environment variable
export OPENAI_API_KEY="sk-..."

# OR config file
echo "sk-..." > ~/.config/openai/api_key
```

The scripts will auto-detect and use whichever API key is available.

### Basic Usage

Run the full pipeline:
```bash
python3 scripts/memory-extract.py
python3 scripts/memory-link.py
python3 scripts/memory-briefing.py
```

Or run individual steps as needed.

## Pipeline Stages

### Stage 1: Extract Facts

**Script:** `memory-extract.py`

Reads from (in priority order):
1. Daily memory files (`{workspace}/memory/YYYY-MM-DD.md`) — today or yesterday
2. Session transcripts (`~/.clawdbot/agents/main/sessions/*.jsonl`)

Extracts structured facts:
- **Type**: decision, preference, learning, commitment, fact
- **Content**: The actual information
- **Subject**: What it's about (auto-detected from context)
- **Confidence**: 0.0-1.0 reliability score

**Output:** `{workspace}/memory/extracted.jsonl` — One JSON fact per line, deduplicated

### Stage 2: Build Knowledge Graph

**Script:** `memory-link.py`

Takes extracted facts and:
- Generates embeddings (if OpenAI key available, else uses keyword similarity)
- Creates bidirectional links between related facts
- Detects contradictions and marks superseded facts
- Auto-generates domain tags from content

**Output:**
- `{workspace}/memory/knowledge-graph.json` — Full graph with nodes and links
- `{workspace}/memory/knowledge-summary.md` — Human-readable summary

### Stage 3: Generate Briefing

**Script:** `memory-briefing.py`

Creates a compact daily briefing loaded at session start.

Combines:
- Personality traits (from SOUL.md if exists)
- User context (from USER.md if exists)
- Active projects (top subjects from recent facts)
- Recent decisions and preferences
- Active todos (from any todos*.md files)

**Output:** `{workspace}/BRIEFING.md` — Under 2000 chars, LLM-generated or template-based

## Wiring Into HEARTBEAT.md

To run automatically, add to your workspace's `HEARTBEAT.md`:

```markdown
# Heartbeat Tasks

## Daily (once per day, morning)
- Run memory extraction: `cd {workspace} && python3 skills/memory-pipeline/scripts/memory-extract.py`
- Build knowledge graph: `cd {workspace} && python3 skills/memory-pipeline/scripts/memory-link.py`
- Generate briefing: `cd {workspace} && python3 skills/memory-pipeline/scripts/memory-briefing.py`

## Weekly (Sunday evening)
- Review `memory/knowledge-summary.md` for insights
- Clean up old daily notes (optional)
```

## Loading BRIEFING.md

**Important:** BRIEFING.md needs to be loaded as workspace context at session start. This requires the OpenClaw context loading feature (currently in development).

Once available, configure your agent to load BRIEFING.md along with SOUL.md, USER.md, and AGENTS.md at the start of each session.

## Output Files

All files are created in `{workspace}/memory/`:

- **extracted.jsonl** — All extracted facts (append-only)
- **knowledge-graph.json** — Full knowledge graph with embeddings and links
- **knowledge-summary.md** — Human-readable summary of the graph
- **BRIEFING.md** (in workspace root) — Daily context cheat sheet

## Customization

### Changing Models

Edit the model names in each script:
- `memory-extract.py`: Lines with `"model": "gpt-4o-mini"` (or claude/gemini equivalents)
- `memory-link.py`: Line with `"model": "text-embedding-3-small"`
- `memory-briefing.py`: Lines with `"model": "gpt-4o-mini"`

### Adjusting Extraction

In `memory-extract.py`, modify the extraction prompt (lines ~75-85) to focus on different types of information or change the output format.

### Link Threshold

In `memory-link.py`, change the similarity threshold for creating links (currently 0.3 at line ~195).

## Troubleshooting

**No facts extracted:**
- Check that daily notes or transcripts exist
- Verify API key is set correctly
- Check script output for LLM errors

**Low-quality links:**
- Add OpenAI API key for embedding-based similarity (more accurate than keyword matching)
- Adjust similarity threshold in `memory-link.py`

**Briefing too long:**
- Reduce number of facts included in template (edit `generate_fallback_briefing`)
- LLM-generated briefings are automatically constrained to 2000 chars

## See Also

- [Setup Guide](references/setup.md) — Detailed installation and configuration
