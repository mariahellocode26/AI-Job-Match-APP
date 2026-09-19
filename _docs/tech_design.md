# AI Job Match Navigator — Python/Streamlit Tech Design

## Context

`_docs/design_doc.md` fully specifies the product (resume parsing → job discovery → strict verification → priority-scored ranking → resume/skills insights), across 5 pages. This document lays out how to build it in Python with Streamlit as the interface.

Two decisions are settled:
- **LLM**: Claude via the Anthropic API (resume parsing, match explanations, verification comparison).
- **Job discovery sources**: "decide later" on the full platform list — so discovery is built as a pluggable interface, and the two source types that work with **zero paid API keys and no ToS risk** are wired up first (Remote OK's public JSON API, and direct Greenhouse/Lever/Ashby company board APIs), so the app is fully runnable end-to-end without waiting on a paid aggregator decision. LinkedIn/Indeed/Glassdoor/Dice/ZipRecruiter are stubbed as "not yet connected" sources — adding them later (via a paid aggregator API such as JSearch or Adzuna) is a matter of dropping in one new module, not restructuring anything.

Goal: a working vertical slice — upload a resume, click Search Jobs, see verified/unverified/ranked results and resume/skills insights — using real (if narrower-than-final) data sources, matching the doc's page structure and data model.

## Architecture

```
app.py                          # Streamlit entry point = Dashboard
pages/
  1_Best_Jobs.py
  2_Resume_Insights.py
  3_Skills_Market.py
  4_Unverified_Jobs.py
  5_Settings.py
core/
  models.py                     # Job, CandidateProfile, MatchScore (pydantic)
  db.py                         # SQLite helpers: init_db(), upsert_job(), get_jobs(), save_profile(), etc.
  llm.py                        # thin Anthropic client wrapper (structured JSON extraction helper)
  resume/
    parser.py                   # PDF text extraction (pdfplumber) + LLM → CandidateProfile
  discovery/
    base.py                     # DiscoverySource ABC: .search(query) -> list[RawListing]
    remoteok.py                 # real: Remote OK JSON API
    ats_boards.py               # real: direct Greenhouse/Lever/Ashby public board APIs
    stubs.py                    # LinkedIn/Indeed/Glassdoor/... — raise NotConnected, listed in Settings as inactive
    pipeline.py                 # orchestrates: generate queries -> run sources -> normalize -> dedupe
  verification/
    verifier.py                 # confirms a job against its official ATS/careers page, stores evidence
  matching/
    scorer.py                   # rule-based Skill/Experience/Role/Location/Freshness/Verification scoring
    explainer.py                # builds the ✓/△ structured explanation from score components
  skills/
    market.py                   # frequency aggregation, skill normalization, learning-priority formula
requirements.txt
.env.example                    # ANTHROPIC_API_KEY=
data/                           # sqlite db + uploaded resumes (gitignored)
```

Data flow mirrors the doc's pipeline (§22/§26/§34): `Search Request → discovery.pipeline → core.db (raw jobs) → verification.verifier → matching.scorer/explainer → skills.market → Streamlit pages read from core.db`.

## Key implementation notes

- **Storage**: SQLite at `data/app.db`, accessed via plain `sqlite3` + small helper functions in `core/db.py` (no ORM needed at this scale). Tables: `jobs`, `candidate_profile`, `search_runs`. This persists across reruns/restarts, unlike `st.session_state` alone.
- **Job model** (`core/models.py`): mirrors §23 exactly — title, company, location, remote_type, seniority, description, requirements, source(s), posted_at, discovered_at, official_posting_url, apply_url, verification_status/source/url/method/timestamp, status, direct_email.
- **Resume parsing**: `pdfplumber` for text extraction; `core/llm.py` calls Claude with a JSON schema prompt to produce the structured `CandidateProfile` (§A2/§A3). Store both raw text and structured profile.
- **Discovery pipeline** (`core/discovery/pipeline.py`): takes preferences → builds search queries → calls each active `DiscoverySource` → normalizes results into `Job` objects → hands off to verification. Adding a new source later = implementing `DiscoverySource.search()` and registering it; nothing else changes (per doc §22/§54's modularity requirement).
- **Verification** (`core/verification/verifier.py`): for ATS-sourced jobs (Greenhouse/Lever/Ashby) verification is close to free — the discovery source *is* the official ATS, so it's marked Verified immediately with `verification_source=ats_direct`. For Remote OK listings, verifier attempts to resolve the company's careers/ATS page and confirm; on failure the job is marked Unverified with a reason, never silently dropped (§5, §25).
- **Scoring** (`core/matching/scorer.py`): hard filters first (§28: closed/removed, >7 days, incompatible location, non-English) then weighted ranking factors using the doc's starting weights (§29: Skills 30/Experience 20/Role 20/Location 15/Freshness 10/Verification 5), configurable via the Settings page. `explainer.py` turns the component scores into the ✓/△ structured text from §31 — no free-form LLM claims.
- **Skills market** (`core/skills/market.py`): implements the normalization groups from §35/§40, frequency-over-relevant-jobs calculation with explicit denominator (§33), and `Learning Priority = Market Demand × Role Relevance × Candidate Gap` (§41) using ordinal buckets (Low/Medium/High/Very High) since these aren't objectively numeric.
- **Email application handling** (§10): tagged `📧 Direct Email Application` when a posting explicitly names an application email; shown inline in Best Jobs plus a dedicated filter. Card renders **email + mailto button** (option B — the doc's own MVP recommendation, since the user left this pending) — noted in-app as a Settings-configurable choice so it's easy to switch to option C later.
- **Settings page**: shows active vs. stubbed discovery sources, lets the user tweak scoring weights, and holds the Anthropic API key status (read from `.env`, never entered/stored in the UI itself).

## Files to create (first build pass)

`app.py`, `pages/1_Best_Jobs.py`, `pages/2_Resume_Insights.py`, `pages/3_Skills_Market.py`, `pages/4_Unverified_Jobs.py`, `pages/5_Settings.py`, `core/models.py`, `core/db.py`, `core/llm.py`, `core/resume/parser.py`, `core/discovery/base.py`, `core/discovery/remoteok.py`, `core/discovery/ats_boards.py`, `core/discovery/stubs.py`, `core/discovery/pipeline.py`, `core/verification/verifier.py`, `core/matching/scorer.py`, `core/matching/explainer.py`, `core/skills/market.py`, `requirements.txt`, `.env.example`, updated `README.md`, `.gitignore` additions for `data/` and `.env`.

## Verification plan (once built)

1. `pip install -r requirements.txt`, copy `.env.example` → `.env` with a real `ANTHROPIC_API_KEY`.
2. `streamlit run app.py` — confirm Dashboard loads with zero-state counts.
3. Upload a sample resume PDF on the Dashboard/Settings → confirm `CandidateProfile` is parsed and persisted (re-open app, profile still there).
4. Set preferences, click **Search Jobs** → confirm jobs are discovered from Remote OK + any Greenhouse/Lever/Ashby company boards queried, verification status is set (not left blank), and results land in Best Jobs (verified) vs. Unverified Jobs pages correctly.
5. Confirm Best Jobs filters (§46) and job cards (§47) render score breakdowns and the ✓/△ explanation.
6. Confirm Skills Market and Resume Insights pages populate from the same search run's job set.
