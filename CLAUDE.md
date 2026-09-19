# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project status

This repository is **pre-implementation**: only planning docs exist so far (`_docs/`). There is no `app.py`, `core/`, `pages/`, `requirements.txt`, or test suite yet — those are created by working through `_docs/tasks.md`. Do not assume any code exists; check before referencing a module.

## Reference docs (read before building)

- `_docs/design_doc.md` — the product spec: what the app does, all page/feature requirements, the data model, scoring/verification rules.
- `_docs/tech_design.md` — the technical architecture: how the product spec maps to Python modules, the Streamlit page layout, and the discovery/verification/matching pipeline.
- `_docs/tasks.md` — the implementation backlog. Each task is scoped to one session and independent enough to work from its own description. Work through these in order unless told otherwise; task 1 sets up the project skeleton and a passing test.

## Commands (once the project is scaffolded per task 1)

```bash
pip install -r requirements.txt   # install dependencies
cp .env.example .env              # then set ANTHROPIC_API_KEY
streamlit run app.py              # run the app
pytest                            # run the full test suite
pytest tests/path_to_test.py::test_name   # run a single test
```

## Architecture (per `_docs/tech_design.md`)

The app is a Streamlit multipage app backed by a plain Python `core/` package and a local SQLite database (`data/app.db`) — no external backend service.

**Pipeline**: `Search Request → core/discovery (pluggable sources) → core/db (raw jobs) → core/verification → core/matching (scorer + explainer) → core/skills (market analysis) → Streamlit pages read from core/db`.

Key structural points:

- **`core/models.py`** defines the shared `Job`, `CandidateProfile`, and `MatchScore` pydantic models used across every other module — start here when touching data shapes.
- **Discovery is pluggable**: every source (`core/discovery/*.py`) implements the `DiscoverySource` interface in `core/discovery/base.py`. Real sources today are Remote OK's public JSON API and direct Greenhouse/Lever/Ashby ATS board APIs (no paid keys needed); LinkedIn/Indeed/Glassdoor/Dice/ZipRecruiter/etc. are intentionally stubbed as "not connected" rather than scraped, because those platforms actively block scraping and have no free API. Adding a new source later means implementing one class and registering it — nothing else in the pipeline changes.
- **ATS-sourced jobs are verified at discovery time** (the source *is* the official ATS); jobs from other sources go through `core/verification/verifier.py`, which independently confirms them against the company's careers page/ATS and always records evidence (`verification_source`, `verification_url`, `verification_method`, `verification_timestamp`) — never silently drops an unconfirmed job, it's marked Unverified instead.
- **Scoring applies hard filters before ranking factors** (`core/matching/scorer.py`): closed/removed/stale/wrong-location/non-English jobs are excluded outright, then remaining jobs get a weighted Application Priority Score (configurable weights, starting point Skills 30% / Experience 20% / Role 20% / Location 15% / Freshness 10% / Verification 5%). This ordering matters — a strong skills match must never override a hard location/status disqualification.
- **All LLM calls go through `core/llm.py`**, a thin Anthropic (Claude) wrapper used for resume parsing into `CandidateProfile`. Match explanations and verification comparisons are built from structured score/evidence data, not free-form LLM output — see `explainer.py`.
- **Never recommend claiming a skill the candidate doesn't have.** Resume keyword and skills-market recommendations (`core/resume/parser.py`, `core/skills/market.py`) must distinguish "already has the experience → emphasize it" from "doesn't have it → learn it first." This is a product safety rule from `_docs/design_doc.md` §37.5, not a general style preference.

## Keeping this file current

Update this file as the project evolves — when commands, structure, or key architectural decisions change (e.g. a new discovery source ships, the storage layer changes, or a task from `_docs/tasks.md` alters the architecture described here).
