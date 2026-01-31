# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Indie Spark** is a daily email workflow that generates cozy game development ideas for solo indie devs. It is not a traditional software project — it is an **n8n workflow automation** consisting of two files:

- `indie-spark-workflow.json` — The complete n8n workflow definition (importable directly into n8n)
- `indie-spark-implementation-plan.md` — Step-by-step implementation guide and architecture reference

## Architecture

The workflow runs daily at 7 AM and follows this pipeline:

```
Schedule Trigger → 6 Parallel Data Fetchers → Merge All → Build Prompt → Gemini 1.5 Flash AI Agent → Process Response → Build HTML Email → Send via SMTP
```

**Data sources** (fetched in parallel with `continueOnFail: true` and hardcoded fallbacks):
- Reddit (r/CozyPlaces, r/TinyHouses) — cozy aesthetic inspiration
- NASA APOD — science/nature hooks
- Project Gutenberg — narrative inspiration from random books
- Itch.io Game Jams — active jam themes
- Hardcoded board game mechanics list (15 mechanics, 3 randomly selected)

**AI configuration:** Gemini 1.5 Flash, temperature 0.9, 3000 max tokens, via LangChain AI Agent node.

## Key Technical Details

- All processing logic lives in n8n **Code nodes** (JavaScript/Node.js runtime)
- The workflow JSON contains 24 nodes total with embedded JavaScript in `jsCode` fields
- Node IDs follow the pattern `fetch-*`, `process-*`, `build-*`, `merge-*`, `send-*`
- Error resilience: every HTTP node has `continueOnFail: true` with fallback data in corresponding process nodes
- Email output uses inline-styled HTML with Georgia serif font and a warm brown/green color palette

## Working with the Workflow JSON

When editing `indie-spark-workflow.json`:
- Node connections are defined in the `connections` object at the bottom, mapping output ports to input ports by node name
- JavaScript code is stored as a single string in `parameters.jsCode` — be careful with escaping
- Node positions (`position: [x, y]`) affect visual layout in the n8n editor but not execution
- The workflow uses both `schedule-trigger` (production) and `manual-trigger` (testing) entry points

## Design Constraints

Game ideas generated must follow these rules (enforced via the AI system prompt):
- No combat, no death, no time pressure, no stress
- Unusual/experimental mechanics or perspectives
- Realistic solo dev scope (1-3 months)
- Each idea must be distinctly different
- Specific descriptions, not generic

## Required Credentials (configured in n8n, not in this repo)

- Gemini API key (HTTP Header Auth)
- SMTP credentials (host, port, user, password)
- Optional: Reddit API for higher rate limits
