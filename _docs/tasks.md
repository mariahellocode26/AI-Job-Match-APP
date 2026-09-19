# AI Job Match Navigator — Implementation Backlog

Each task below is scoped to one working session and is self-contained: it names the files it touches and enough context to start without reading the other tasks. Background/architecture reference: `_docs/design_doc.md` (product spec) and `_docs/tech_design.md` (technical design).

---

## 1. Project scaffold with a passing test
Goal: Get an empty, installable project running with a green test suite.
Description: Create the repo skeleton (`app.py` stub, `core/` package with `__init__.py`, `requirements.txt` with `streamlit`, `pytest`, and `pydantic`, `.env.example` with `ANTHROPIC_API_KEY=`, `.gitignore` entries for `data/` and `.env`). Add a `tests/` folder with one trivial test (e.g. asserting the package imports cleanly) and confirm `pytest` passes and `streamlit run app.py` launches a blank page.

## 2. Core data models
Goal: Define the shared data structures every other module will build on.
Description: In `core/models.py`, implement `Job`, `CandidateProfile`, and `MatchScore` as pydantic models. `Job` should include every field listed in design_doc.md §23 (title, company, location, remote_type, seniority, description, requirements, source, source_url, posted_at, discovered_at, official_posting_url, apply_url, verification_status, verification_source, status, direct_email). Add unit tests that construct valid/invalid instances of each model.

## 3. SQLite storage layer
Goal: Persist jobs, candidate profiles, and search runs across app restarts.
Description: In `core/db.py`, implement `init_db()`, and CRUD helpers (`upsert_job`, `get_jobs`, `save_profile`, `get_profile`, `save_search_run`) backed by plain `sqlite3` against `data/app.db`, with tables for `jobs`, `candidate_profile`, and `search_runs`. Write tests using a temporary SQLite file to confirm round-tripping of a `Job` and a `CandidateProfile`.

## 4. Resume PDF text extraction
Goal: Turn an uploaded resume PDF into raw text.
Description: In `core/resume/parser.py`, add a function that takes PDF bytes/path and returns extracted plain text using `pdfplumber`, handling multi-column and multi-page layouts reasonably. Include a small sample resume PDF under `tests/fixtures/` and a test asserting key known strings (name, a skill) are present in the extracted text.

## 5. Anthropic LLM client wrapper
Goal: Provide one shared, testable entry point for all Claude calls.
Description: In `core/llm.py`, implement a thin wrapper around the Anthropic Python SDK with a function like `extract_json(prompt, schema_description) -> dict` that sends a prompt instructing Claude to return structured JSON and parses/validates the response. Read `ANTHROPIC_API_KEY` from the environment. Write a test that mocks the Anthropic client so no real API call is made.

## 6. Resume → structured candidate profile
Goal: Convert extracted resume text into a `CandidateProfile`.
Description: In `core/resume/parser.py`, add a function that takes raw resume text, calls `core/llm.py`'s `extract_json` with a prompt describing the fields from design_doc.md §11/§A2 (roles, years of experience, skills, technologies, education, AI/ML and cloud experience), and returns a populated `CandidateProfile` (from task 2). Test with a mocked LLM response mapping to a known profile.

## 7. Discovery source interface
Goal: Define the pluggable contract every job source must implement.
Description: In `core/discovery/base.py`, define a `DiscoverySource` abstract base class with a `search(query: SearchQuery) -> list[RawListing]` method and a small `RawListing` structure for not-yet-normalized results. Add a fake in-memory test source implementing the interface, and a test confirming the contract (return type, required fields) is enforced.

## 8. Remote OK discovery source
Goal: Pull live listings from Remote OK's public JSON API.
Description: In `core/discovery/remoteok.py`, implement `DiscoverySource` (from task 7) by calling Remote OK's public JSON API, filtering to relevant/software roles, and mapping each result into a `Job` (task 2) with `source="remote_ok"`. Mock the HTTP call in tests and assert correct normalization of a sample API response.

## 9. Direct ATS discovery sources (Greenhouse / Lever / Ashby)
Goal: Pull listings directly from a company's official ATS board.
Description: In `core/discovery/ats_boards.py`, implement functions/classes that query the public Greenhouse, Lever, and Ashby job-board APIs for a given company slug and normalize results into `Job` objects with `source` set to the ATS name. Since the source *is* the official ATS, mark these `verification_status="verified"` and `verification_source="ats_direct"` at creation time. Mock HTTP responses in tests for each of the three ATS types.

## 10. Stubbed discovery sources
Goal: Represent not-yet-connected platforms without breaking the pipeline.
Description: In `core/discovery/stubs.py`, add placeholder `DiscoverySource` implementations for LinkedIn, Indeed, Glassdoor, Wellfound, Built In, We Work Remotely, Welcome to the Jungle, Dice, and ZipRecruiter that raise a clear `SourceNotConnected` exception (or return an empty list with a "not connected" flag) rather than attempting to scrape them. Write a test confirming the pipeline can list these sources as inactive without erroring.

## 11. Discovery pipeline orchestrator
Goal: Turn user preferences into a deduplicated list of normalized jobs.
Description: In `core/discovery/pipeline.py`, implement a function that takes the four MVP preferences (target roles, seniority, work arrangement, target locations), builds search queries, calls every active `DiscoverySource`, merges results, and deduplicates reposts into one canonical `Job` per design_doc.md §7 (keeping other sources as metadata). Test using fake sources from task 7 with overlapping/duplicate listings.

## 12. Job verification module
Goal: Independently confirm a discovered job against its official source.
Description: In `core/verification/verifier.py`, implement a function that, given a `Job` not already ATS-verified, attempts to locate the company's careers page or ATS listing, compares company/title/core details using flexible matching (design_doc.md §5), and sets `verification_status` to `verified`/`unverified` plus the evidence fields from §25 (`verification_source`, `verification_url`, `verification_method`, `verification_timestamp`). Test with mocked HTTP lookups for a matching and a non-matching case.

## 13. Hard filters and scoring engine
Goal: Compute the Application Priority Score for a verified job.
Description: In `core/matching/scorer.py`, implement hard filters (closed/removed, older than 7 days, incompatible location, non-English, per design_doc.md §28) applied before a weighted scoring function using the starting weights from §29 (Skills 30%, Experience 20%, Role 20%, Location 15%, Freshness 10%, Verification 5%), taking a `Job` and `CandidateProfile` and returning a `MatchScore`. Make weights a configurable parameter, not hardcoded. Write tests covering a filtered-out job and a fully-scored job.

## 14. Match explanation generator
Goal: Produce the human-readable "why this matches" breakdown.
Description: In `core/matching/explainer.py`, implement a function that takes a `MatchScore` and its component evidence and produces the structured ✓/△ explanation format from design_doc.md §31 (strong points, then potential gaps), built entirely from structured data rather than free-form LLM text. Test with a sample score object asserting the expected checklist lines appear.

## 15. Skills market analysis
Goal: Aggregate in-demand skills across a set of relevant jobs and compare to the candidate.
Description: In `core/skills/market.py`, implement skill normalization (grouping synonyms like "LLMs"/"Generative AI" per §35/§40), frequency calculation with an explicit denominator (§33), and a `Learning Priority = Market Demand × Role Relevance × Candidate Gap` computation using ordinal buckets (Low/Medium/High/Very High) per §41. Test with a small fixed set of jobs and a candidate profile, asserting expected frequencies and priorities.

## 16. Dashboard page
Goal: Give the user a high-level landing page with navigation.
Description: In `app.py`, build the Streamlit Dashboard showing the summary metrics from design_doc.md §15 (Verified Jobs, 🔥 New, 🟢 Recent, 📧 Direct Email, ⭐ Top Opportunities counts) read from `core/db.py`, plus a resume-upload widget wired to tasks 4/6, and links to the other pages. No live search triggering here.

## 17. Best Jobs page
Goal: Show ranked, verified jobs with filters.
Description: In `pages/1_Best_Jobs.py`, render all verified jobs from the last 7 days ranked by Application Priority Score, with filters for role, seniority, location, work arrangement, minimum score, freshness, and direct-email-only (§46), and job cards matching the §47 layout (score, badges, skill/experience/role/location breakdown, both application links). Top 10-20 should be visually emphasized.

## 18. Resume Insights page
Goal: Show the candidate keyword/skill analysis.
Description: In `pages/2_Resume_Insights.py`, render Strong Matching Skills, Missing High-Impact Keywords, Underrepresented Keywords, and Recommendations sections (§17, §37) using the candidate profile and the most recent search run's job set, following the safety rule that recommendations never suggest claiming a skill the candidate doesn't have (§37.5).

## 19. Skills Market page
Goal: Show market-wide skill demand and learning priorities.
Description: In `pages/3_Skills_Market.py`, render a table of Skill / Demand / Candidate Status / Learning Priority (§18/§50) sourced from `core/skills/market.py` (task 15) for the most recent search run.

## 20. Unverified Jobs page
Goal: Show relevant jobs that could not be officially confirmed, separately from the main rankings.
Description: In `pages/4_Unverified_Jobs.py`, list jobs with `verification_status="unverified"` that passed freshness/relevance filters, each showing the 🟡 Unverified tag and the reason from `core/verification/verifier.py` (task 12), per design_doc.md §19/§51. Confirm these never appear in Best Jobs' ranked list.

## 21. Settings page
Goal: Let the user see and tune the parts of the system that are configurable.
Description: In `pages/5_Settings.py`, show which discovery sources are active vs. stubbed (task 10), let the user adjust the scoring weights from task 13 (persisted via `core/db.py`), and show whether `ANTHROPIC_API_KEY` is configured (without letting it be typed into the UI).

## 22. End-to-end "Search Jobs" wiring
Goal: Connect the full pipeline to the manual search button.
Description: Wire a "Search Jobs" action (on the Dashboard or Best Jobs page) that reads the candidate profile and preferences, and runs discovery (task 11) → verification (task 12) → scoring (task 13) → explanation (task 14) → skills market (task 15), persisting results via `core/db.py` and refreshing the affected pages, per the pipeline order in design_doc.md §52. Add an integration test that runs the full pipeline with fake/mocked sources and asserts jobs land in the correct DB state.

## 23. Setup docs and run instructions
Goal: Make the project easy for a new contributor to run.
Description: Update `README.md` with install steps (`pip install -r requirements.txt`), `.env` setup, how to run (`streamlit run app.py`) and how to run tests (`pytest`), plus a short note on how to add a new discovery source (task 7/10) so the modularity described in `_docs/tech_design.md` is discoverable to newcomers.
