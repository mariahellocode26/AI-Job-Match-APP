---
name: job-match-prototype
description: Prototype job search skill for AI/software engineering roles. Discovers recent postings from zero-risk sources (direct ATS APIs, free job APIs, web search, optional LinkedIn via Apify) and independently verifies each one against the company's official careers page or ATS before calling it Verified. Use when the user says "find jobs", "search jobs", "job search", or invokes /job-match-prototype.
---

# Job Match Prototype

This is an experimental prototype (see `_docs/design_doc.md` for the full product vision and `_docs/tech_design.md` for the planned Streamlit app this prototype is validating). It exists to answer one question: **can live discovery + independent verification, run conversationally, actually find good confirmed jobs?** It is deliberately narrower than the full product — no resume PDF parsing, no weighted Application Priority Score. Its job is discovery + verification, done rigorously.

## The one non-negotiable rule

**Never label a job "Verified" without actually fetching and confirming it against the company's official careers page or ATS.** A job found via search or a scraper is a *candidate*, not a verified result, until that confirmation step succeeds. If verification can't be completed, the job goes in the Unverified section with a stated reason — it is never dropped silently and never blended into the Verified list. This applies equally to every source, LinkedIn included — do not auto-verify a LinkedIn/Apify result just because it came from a paid, structured source.

## Step 1 — Gather preferences

If not already stated in the conversation, ask the user for these four (per design_doc.md §3):
1. Target roles (e.g. "AI Engineer", "Backend Engineer")
2. Seniority (e.g. Mid-level, Senior)
3. Work arrangement (Remote / Hybrid / Onsite)
4. Target locations (e.g. EMEA, Egypt, specific countries)

Also confirm the candidate's location-compatibility constraints from design_doc.md §2: remote from EMEA, remote from North America/Canada, worldwide-where-allowed, or onsite/hybrid in Egypt. US-only roles (no explicit EMEA/international remote support and no Egypt/EMEA hiring presence) should be excluded. Ambiguous remote geography should be flagged, not silently included. Work authorization should only be evaluated when a posting explicitly states a requirement relevant to the candidate's locations — never infer visa/authorization status from silence.

## Step 2 — Discover candidate jobs

Use these sources, in this priority order:

1. **Direct ATS APIs (no scraping, fully public)** — for any company you know or discover is hiring, fetch directly:
   - Greenhouse: `https://boards-api.greenhouse.io/v1/boards/{company-slug}/jobs`
   - Lever: `https://api.lever.co/v0/postings/{company-slug}?mode=json`
   - Ashby: `https://api.ashbyhq.com/posting-api/job-board/{company-slug}`
   Since the source *is* the official ATS, a job found this way can skip straight to Verified with `verification_source: ats_direct` — no separate confirmation fetch needed.

2. **Free job APIs** — fetch `https://remoteok.com/api` (public JSON) and filter to relevant roles.

3. **Web search** — use WebSearch with queries built from the target roles/seniority/locations (e.g. `"AI Engineer" remote EMEA jobs`, `site:boards.greenhouse.io AI engineer`) to surface both direct postings and company names/slugs to feed back into step 1.

4. **LinkedIn, only if the user has explicitly opted in and an Apify (or similar) MCP tool is available in this session** — check for a connected LinkedIn-scraping tool before attempting anything. If none is available, skip this source with a one-line note ("LinkedIn source not configured — connect an Apify MCP server to enable it") rather than attempting to fetch or scrape linkedin.com directly, which is blocked and against its ToS. If it is available and used, tag every result from it clearly (see Output format) — this source carries real ToS/legal risk that the user has knowingly accepted, and results from it get no special treatment in verification.

**Do not** write or attempt any first-party scraping code against LinkedIn, Indeed, Glassdoor, Dice, or ZipRecruiter — no direct HTTP fetch of pages on those domains for the purpose of extracting listings.

## Step 3 — Normalize

For every candidate listing, extract: title, company, location, remote_type, seniority (best-effort), description/requirements summary, posted_at (actual date, never guessed), source, source_url, apply_url, and direct_email (only if the posting explicitly states an email is an application channel, e.g. "Send your CV to jobs@company.com" — a generic recruiter contact does not count, per design_doc.md §10).

## Step 4 — Filter (hard filters, not scoring)

Drop (do not show at all) any job that is:
- Posted more than 7 days ago, or has no discoverable posting date.
- Non-English.
- Explicitly closed / no longer accepting applications.
- Explicitly US-only, when the candidate isn't eligible per the Step 1 location rules.
- Explicitly restricted to a country the candidate doesn't reside in or isn't covered by the stated remote-work compatibility.

Tag freshness on everything that survives: `<24h` → 🔥 New, `1-7 days` → 🟢 Recent.

## Step 5 — Deduplicate

If the same job is found via more than one source (e.g. Remote OK + a search result + the company's own Greenhouse board), merge into one canonical entry. Prefer the official/ATS posting as canonical, list every other source as metadata (per design_doc.md §7).

## Step 6 — Verify

For every job not already ATS-direct-verified in Step 2:
1. WebFetch the company's careers page (search for it if the URL isn't already known) or its ATS board.
2. Look for the exact job using flexible matching — company must match, title must match, core details (seniority, location, role) must match; minor wording/location phrasing differences are fine (design_doc.md §5).
3. If found and active → `verification_status: verified`, record `verification_source`, `verification_url`, `verification_method` (e.g. "exact title + company + location match on official Greenhouse board"), `verification_timestamp` (now).
4. If not found, or the company's official presence can't be located → `verification_status: unverified`, with a one-line reason (e.g. "Official posting could not be confirmed on company careers page").

## Step 7 — Output

Present two clearly separated sections, each ordered by freshness (🔥 New first, then 🟢 Recent):

### 🟢 Verified Jobs
For each: title, company, location/remote type, freshness tag, source(s) found on, official posting URL, apply URL (if different), direct email tag if applicable, and the verification evidence (source + URL + method).

### 🟡 Unverified Jobs
For each: title, company, location/remote type, freshness tag, source(s) found on, and the reason verification failed. State plainly these are not confirmed and should be treated with more caution.

If any result came from LinkedIn/Apify, prefix it with `🔗 LinkedIn (via Apify)` in both sections so its provenance is never ambiguous.

Do not compute or display an overall priority/match score in this prototype — that belongs to the full app's scoring model (design_doc.md §29), not this discovery/verification test.
