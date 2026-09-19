# AI Job Match Navigator — Complete MVP Scope

## 1. Product Concept

Working name: **AI Job Match Navigator**

Purpose:

- Upload resume PDF.
- Ask for a small set of essential preferences.
- Discover recent AI/software engineering jobs as broadly as possible.
- Use a curated list of trusted job platforms plus search engine/API discovery.
- Strictly verify exact jobs against company official careers pages or official ATS.
- Rank all verified jobs by an **Application Priority Score**, not a claimed hiring probability.
- Highlight top 10–20 best opportunities.
- Show unverified jobs separately.
- Provide resume keyword recommendations.
- Analyze market-demand skills and learning priorities.
- Tag jobs that explicitly allow direct application by email.
- Keep both official job posting URL and direct apply URL.
- Search is manual only in MVP.

---

# 2. Candidate / Location Assumptions

The candidate can:

- Work remotely from EMEA.
- Work remotely from North America, including Canada.
- Work worldwide where allowed.
- Work onsite/hybrid in Egypt.

US companies are especially valuable if they:

- Explicitly support EMEA/international remote work, OR
- Have an established Egypt/EMEA entity or hiring presence.

US-only roles should be excluded.

Remote roles without explicit EMEA/international compatibility should not be treated as verified compatible recommendations.

Country-specific remote roles are not automatically compatible if the candidate does not reside in that country.

Unclear remote geography should get a warning/lower priority.

Explicit incompatible country restrictions should lower priority or exclude.

Work authorization should only be evaluated when explicitly stated in the posting and relevant to candidate locations; do not infer visa status.

---

# 3. User Preferences in MVP

Only essential preferences:

1. Target roles
2. Seniority
3. Work arrangement: Remote / Hybrid / Onsite
4. Target locations: EMEA / Egypt / specific countries

References are explicitly removed from MVP.

No salary preference was added.

---

# 4. Job Discovery

The user chose:

- **As broad as possible** discovery.
- But discovery should use a curated trusted platform allowlist rather than dynamically trusting arbitrary platforms.
- Search engine discovery + available job APIs.
- No broad scraping dependency.
- English only.
- Manual search only: user clicks **Search Jobs**.
- No scheduled/background monitoring in MVP.

### Initial Trusted Platform Allowlist

1. LinkedIn
2. Indeed
3. Glassdoor
4. Wellfound
5. Built In
6. Remote OK
7. We Work Remotely
8. Welcome to the Jungle
9. Dice
10. ZipRecruiter

Important principle:

> Trusted platform ≠ every individual job is legitimate.

Platforms are accepted discovery sources; individual jobs still need independent verification.

---

# 5. Verification

Strictest verification:

- Exact job must be independently confirmed on company official careers site or official ATS.
- Official ATS can count if relationship to company is reliably established.

Supported major ATS examples:

- Greenhouse
- Lever
- Ashby
- Workday
- SmartRecruiters
- iCIMS
- Jobvite

### ATS Relationship Rule

The company website should link to the job/ATS OR the company-to-ATS relationship can be independently established.

### Job Board vs Official Posting Comparison

Use flexible matching:

- Company should match.
- Title should match.
- Core job details should match.
- Minor wording/location differences are allowed.

### Job Status

- Active / accepting applications → **Verified**
- Status unclear → **Unverified**
- Closed / removed → **Excluded**

If the exact official job cannot be found:

- Show it separately as **Unverified**.
- Do not include it in primary Verified recommendations.

---

# 6. Job Freshness

Only jobs posted within the last **7 days**.

Jobs within the last 24 hours get higher priority/freshness tagging.

Rules:

- `<24h` → higher priority
- `1–7 days` → recent
- `>7 days` → excluded

Reposts should be deduplicated into one job using the newest active posting date.

### Suggested Tags

- `<24h` → `🔥 New`
- `1–7 days` → `🟢 Recent`

---

# 7. Duplicates

Show one canonical job and retain discovery sources as metadata.

Example:

```text
AI Engineer — Company X

Found on:
- LinkedIn
- Indeed
- Wellfound
- Company Careers
```

Official posting = canonical source.

Official apply URL = canonical application destination.

---

# 8. Application Links

Keep both:

1. **Official job posting URL**
2. **Direct application URL**

They may be different.

Verified jobs should expose both separately.

---

# 9. Ranking

The user chose:

* **D + C:** overall Application Priority Score with transparent individual factors.
* All verified jobs from the last 7 days should be shown, ranked.
* Top 10–20 highlighted as best opportunities.
* Unverified jobs separate and do not affect primary recommendations.

### Suggested Factors

* Resume/job match
* Skills match
* Experience match
* Role match
* Location compatibility
* Freshness
* Verification

Example:

```text
AI Engineer — Company X

Application Priority: 94/100

Resume/Job Match: 91%
Skills: 94%
Experience: 95%
Location: 100%
Freshness: <24h
Verification: Official
```

Important:

> Do NOT call this a true "probability of success" or hiring probability.

Use:

* Application Priority Score
* Strong Match
* Match Strength
* Recommended Priority

---

# 10. Direct Email Application Feature

A separate category/list is available for jobs with direct application email.

### Rules

Only include the email if the posting explicitly says it is an application channel, for example:

> Send your CV to jobs@company.com

Generic recruiter/contact email does not count.

Tag:

```text
📧 Direct Email Application
```

No separate scoring.

Use the same Application Priority Score.

The user chose **A**:

> Email-application jobs should appear normally in the main Verified Jobs list with the tag, and also be available in a separate filtered/list view.

### Pending Decision

The assistant asked what the email application card should show:

* A. Email only
* B. Email + button to open email client
* C. Email + suggested subject + suggested email draft
* D. B + C

The assistant recommended **B** for MVP.

The user did **not** answer this question.

Therefore this remains **Pending Decision**.

AI-generated application emails remain out of scope for MVP.

---

# 11. Resume Parsing / Analysis

Input:

> Resume PDF

Parse:

* Target roles / likely roles
* Years of experience
* Skills
* Experience areas
* Projects/technologies
* Job titles
* Education
* AI/ML experience
* Backend/software engineering
* Cloud/DevOps

Generate a structured candidate profile.

---

# 12. Resume Keyword Recommendations

The user chose:

> **B = keywords + recommendation**

Not a full resume rewrite.

Output example:

```text
LLM Evaluation

Found in:
46% of relevant AI Engineer postings.

Recommendation:
Your resume mentions evaluation indirectly.
If supported by your experience, explicitly mention
LLM evaluation in your RAG project.
```

Important:

* Never encourage false skills.
* Only recommend explicitly naming a keyword if the candidate genuinely has supporting experience.
* Can suggest where/how to emphasize existing experience.

---

# 13. Market-Demand Skills

Aggregate requirements across relevant job postings.

Suggested formula:

```text
Learning Priority
=
Market Demand
×
Relevance to Target Role
×
Candidate Skill Gap
```

Potential output fields:

* Skill
* Frequency across postings
* Present in resume? yes/no
* Learning priority
* Why it matters

Example:

```text
LangGraph

Market demand: Medium-High
Candidate gap: Yes
Target-role relevance: Very High
Learning Priority: Very High
```

---

# 14. Suggested Application Pages

Primary pages:

1. Dashboard
2. Best Jobs
3. Resume Insights
4. Skills Market
5. Unverified Jobs
6. Settings (optional)

---

# 15. Dashboard

Possible dashboard metrics:

```text
Verified Jobs          124
🔥 New Jobs             18
🟢 Recent Jobs         106
📧 Direct Email Jobs    11
⭐ Top Opportunities    20
```

The dashboard should allow the user to quickly navigate to the most important results.

---

# 16. Best Jobs

The Best Jobs page contains all **Verified Jobs** from the last 7 days.

Jobs are ranked by:

> **Application Priority Score**

The Top 10–20 opportunities should receive stronger visual emphasis.

---

# 17. Resume Insights

Show:

### Strong Matching Skills

Skills and experience that align well with target jobs.

### Missing High-Impact Keywords

Keywords frequently appearing in relevant jobs but absent from the resume.

### Underrepresented Keywords

Keywords for which the candidate has relevant experience but does not explicitly emphasize it.

### Recommendations

Specific actions the candidate can take to improve the discoverability of existing experience.

---

# 18. Skills Market

Show:

* Most requested skills
* Demand frequency
* Candidate skill status
* Candidate skill gaps
* Learning priorities

Example:

| Skill      | Demand      | Candidate | Priority  |
| ---------- | ----------- | --------- | --------- |
| Python     | Very High   | Strong    | Low       |
| AWS        | High        | Strong    | Low       |
| Kubernetes | High        | Gap       | High      |
| LangGraph  | Medium-High | Gap       | Very High |
| Terraform  | Medium      | Gap       | Medium    |

---

# 19. Unverified Jobs

The Unverified Jobs page contains jobs that:

* Came from an approved discovery platform.
* Meet the freshness requirement.
* Appear relevant.
* Could not be officially confirmed.

Each should clearly display:

```text
🟡 Unverified
```

Example reason:

```text
Official posting could not be confirmed.
```

Unverified jobs should not be part of the primary verified ranking.

---

# 20. Complete MVP Feature Scope

## A. Resume Processing

### A1. Upload Resume

* Accept PDF resume.
* Extract text.
* Handle common resume layouts.
* Store the original resume and extracted text.

### A2. Resume Parsing

Extract:

* Contact/basic information where needed
* Job titles
* Years of experience
* Skills
* Technologies
* Projects
* Education
* AI/ML experience
* Backend experience
* Cloud/DevOps experience

### A3. Candidate Profile

Create a normalized structured profile that can be reused across job searches.

---

# 21. Job Search Preferences

The preferences model should remain intentionally small.

```text
Target Roles
Seniority
Work Arrangement
Target Locations
```

These preferences drive:

* Search queries
* Job filtering
* Matching
* Ranking
* Market-skill analysis

---

# 22. Job Discovery Pipeline

The discovery pipeline should be modular.

```text
Search Request
      |
      ↓
Generate Search Queries
      |
      ↓
Search Trusted Platforms
      |
      ↓
Search Engine Discovery
      |
      ↓
Job APIs
      |
      ↓
Collect Listings
      |
      ↓
Normalize Jobs
```

Each discovered listing should be converted into a common internal job format.

---

# 23. Normalized Job Object

A normalized job should contain information such as:

```text
Job
├── title
├── company
├── location
├── remote_type
├── seniority
├── description
├── requirements
├── responsibilities
├── source
├── source_url
├── posted_at
├── discovered_at
├── official_posting_url
├── apply_url
├── verification_status
├── verification_source
├── status
└── direct_email
```

---

# 24. Job Verification Pipeline

```text
Discovered Job
      |
      ↓
Identify Company
      |
      ↓
Find Official Careers Site
      |
      ↓
Find Official ATS
      |
      ↓
Search Exact Job
      |
      ↓
Compare Details
      |
      ↓
Check Active Status
      |
      +-------------------------+
      |                         |
      ↓                         ↓
   Verified                 Unverified
```

---

# 25. Verification Evidence

For each verified job, store the evidence used for verification.

Possible fields:

```text
verification_status
verification_source
verification_url
verification_method
verification_timestamp
```

Example:

```text
verification_status:
verified

verification_source:
company_careers

verification_url:
official careers URL

verification_method:
exact title + company + location match
```

This makes the system auditable.

---

# 26. Job Matching Pipeline

After verification:

```text
Verified Job
     |
     ↓
Structured Job Requirements
     |
     ↓
Candidate Profile
     |
     +----------------------+
     |                      |
     ↓                      ↓
Rule-Based Matching    Semantic Matching
     |                      |
     +----------+-----------+
                |
                ↓
        Score Calculation
                |
                ↓
       Explanation Generation
```

---

# 27. Matching Dimensions

The system should separately evaluate:

## Role Match

Does the job match the candidate's desired role?

## Skill Match

How many relevant required/preferred skills are supported?

## Experience Match

Does the candidate's experience level align with the role?

## Location Match

Can the candidate realistically work from the specified location?

## Freshness

How recently was the job posted?

## Verification

Is the job officially confirmed?

---

# 28. Hard Filters vs Ranking Factors

Some conditions should be **hard filters**, while others should be ranking factors.

## Hard Filters

Examples:

* Closed job
* Removed job
* Older than 7 days
* Explicitly incompatible location
* US-only when the candidate is not eligible
* Non-English job when English-only search is selected

## Ranking Factors

Examples:

* Skills match
* Experience match
* Role match
* Freshness
* Location strength
* Semantic similarity

This prevents an excellent skill match from overriding a fundamental location incompatibility.

---

# 29. Application Priority Score — Suggested Model

A potential initial scoring model:

```text
Application Priority Score
=
Skill Match
+ Experience Match
+ Role Match
+ Location Compatibility
+ Freshness
+ Verification
```

The exact weights should be configurable and tested.

For example, a future implementation might use:

```text
Skills Match          30%
Experience Match      20%
Role Match            20%
Location              15%
Freshness             10%
Verification           5%
```

These weights are a **starting hypothesis**, not a final product decision.

The score should be calibrated using real job data and user feedback.

---

# 30. Score Interpretation

The score should not be described as:

* Probability of getting hired
* Probability of passing ATS
* Probability of receiving an interview

Instead use:

* Application Priority Score
* Match Strength
* Recommended Priority

Example:

```text
94/100 — Strong Match
```

---

# 31. Match Explanation

Every highly ranked job should have a concise explanation.

Example:

```text
Why this is a strong match:

✓ Strong Python/backend experience
✓ Relevant RAG and LLM experience
✓ PostgreSQL experience matches the requirements
✓ AWS experience matches the cloud requirements
✓ Role aligns with AI Engineer target
✓ Remote EMEA location is compatible

Potential gap:

△ No explicit LangGraph experience found
```

The explanation should be based on structured evidence rather than generic LLM-generated claims.

---

# 32. Resume Keyword Analysis — Detailed Scope

The system should analyze keywords at three levels:

### Level 1 — Present

Keyword exists in the resume.

### Level 2 — Underrepresented

Candidate has related experience, but the keyword is not clearly or explicitly stated.

### Level 3 — Missing

Keyword appears frequently in relevant jobs but no supporting experience was found in the resume.

Example:

```text
RAG

Status:
Present

Recommendation:
No action required.
```

```text
LLM Evaluation

Status:
Underrepresented

Recommendation:
Make the existing evaluation work more explicit.
```

```text
LangGraph

Status:
Missing

Recommendation:
Do not add to resume unless you gain genuine experience.
Consider learning it if demand remains high.
```

---

# 33. Keyword Frequency

Keyword frequency should be calculated against the relevant job dataset.

Example:

```text
LLM Evaluation
46% of relevant jobs

RAG
42% of relevant jobs

LangGraph
34% of relevant jobs
```

The system should specify the denominator:

```text
46% of 120 relevant jobs
```

rather than presenting percentages without context.

---

# 34. Market Skill Analysis Pipeline

```text
Verified + Relevant Jobs
          |
          ↓
Extract Skills
          |
          ↓
Normalize Skill Names
          |
          ↓
Group Related Skills
          |
          ↓
Calculate Frequency
          |
          ↓
Compare With Candidate
          |
          ↓
Calculate Learning Priority
```

---

# 35. Skill Normalization

Different job postings may use different names for related concepts.

Examples:

```text
Large Language Models
LLMs
Generative AI
GenAI
```

Similarly:

```text
Amazon Web Services
AWS
```

can be grouped.

But the system should avoid incorrectly merging distinct technologies.

---

# 36. Candidate Skill Status

Each skill should have a status such as:

* Strong
* Intermediate
* Basic
* Mentioned
* Not found
* Unknown

The system should avoid pretending it can perfectly infer proficiency from a resume.

---

# 37. Resume Keyword Analysis

The system should compare the candidate's resume against relevant job postings.

Conceptually:

```text
Candidate Resume
       |
       ↓
Extract Skills & Experience
       |
       ↓
Relevant Job Requirements
       |
       ↓
Keyword / Skill Comparison
       |
       +-------------------+
       |                   |
       ↓                   ↓
 Existing Keywords     Missing Keywords
       |                   |
       +---------+---------+
                 |
                 ↓
          Recommendations
```

## 37.1 Existing Keywords

Identify important keywords and skills that are already represented in the resume.

Examples:

* Python
* RAG
* LLM
* OpenAI API
* PostgreSQL
* Docker
* AWS
* Vector Search
* Prompt Engineering

## 37.2 Missing Keywords

Identify relevant keywords that:

* Frequently appear in target job postings.
* Are relevant to the candidate's target roles.
* Are not explicitly represented in the resume.

Example:

```text
Keyword:
LLM Evaluation

Market frequency:
46% of relevant AI Engineer postings

Resume:
Not explicitly mentioned
```

## 37.3 Underrepresented Keywords

A keyword may already exist in the resume but not be emphasized strongly enough.

Example:

```text
Keyword:
RAG Evaluation

Resume:
Mentioned indirectly

Recommendation:
Make the evaluation work more explicit in the project description.
```

## 37.4 Keyword Recommendations

Each recommendation should contain:

* Keyword
* Frequency across relevant jobs
* Resume status
* Relevance
* Recommendation
* Reason

Example:

```text
LLM Evaluation

Found in:
46% of relevant AI Engineer jobs

Resume status:
Not explicitly represented

Recommendation:
If your experience includes evaluating RAG/LLM systems,
explicitly mention "LLM evaluation" when describing that work.

Why:
The keyword appears frequently in relevant roles and can make
existing experience easier for recruiters and ATS systems to identify.
```

## 37.5 Important Safety Rule

The system must **never encourage the candidate to claim a skill they do not actually have**.

Recommendations should distinguish between:

```text
Already have the experience
        ↓
Emphasize it
```

and:

```text
Do not have the experience
        ↓
Learn it first
```

Example:

> "Add LangGraph to your resume"

should **not** be recommended unless the candidate genuinely has LangGraph experience.

Instead:

> "LangGraph appears frequently in target roles. Consider learning it."

---

# 38. Resume Improvement Scope

The MVP provides:

* Keyword recommendations
* Keyword frequency
* Missing keywords
* Underrepresented keywords
* Recommendations for emphasizing existing experience

The MVP does **not** automatically rewrite the complete resume.

---

# 39. Market Skills Analysis

The system should aggregate requirements across relevant job postings.

The purpose is to identify what skills employers are actually requesting for the candidate's target roles.

Example:

| Skill      | Job Frequency |
| ---------- | ------------: |
| Python     |           84% |
| AWS        |           67% |
| Docker     |           61% |
| Kubernetes |           48% |
| LangChain  |           45% |
| RAG        |           42% |
| Terraform  |           38% |
| LangGraph  |           34% |

The percentages should be calculated from the actual relevant job dataset.

---

# 40. Market Skill Categories

Skills can be grouped into categories such as:

### Programming

* Python
* Java
* JavaScript
* TypeScript

### AI / ML

* Machine Learning
* Deep Learning
* NLP
* Generative AI
* LLMs

### LLM Engineering

* Prompt Engineering
* Function Calling
* Tool Calling
* Agents
* RAG
* Evaluation
* Guardrails

### Data / Search

* SQL
* PostgreSQL
* Vector Databases
* Embeddings
* Semantic Search
* BM25
* Hybrid Search

### Cloud

* AWS
* Azure
* GCP

### DevOps

* Docker
* Kubernetes
* CI/CD
* Terraform

---

# 41. Skill Learning Priority

The tool should answer:

> **What should I learn next to become more competitive for the jobs I want?**

A suggested conceptual formula is:

```text
Learning Priority
=
Market Demand
×
Target Role Relevance
×
Candidate Skill Gap
```

---

# 42. Skill Learning Example

```text
LangGraph

Market Demand: Medium-High
Target Role Relevance: Very High
Candidate Gap: Yes

Learning Priority:
Very High
```

Another example:

```text
Python

Market Demand: Very High
Target Role Relevance: Very High
Candidate Gap: No

Learning Priority:
Low
```

This prevents the system from recommending skills the candidate already possesses.

---

# 43. Main Application Pages

The MVP should have the following primary pages:

1. Dashboard
2. Best Jobs
3. Resume Insights
4. Skills Market
5. Unverified Jobs
6. Settings (optional)

---

# 44. Dashboard

The dashboard should provide a high-level overview.

Example:

```text
Verified Jobs          124
🔥 New Jobs             18
🟢 Recent Jobs         106
📧 Direct Email Jobs    11
⭐ Top Opportunities    20
```

The dashboard should allow the user to quickly navigate to the most important results.

---

# 45. Best Jobs Page

The Best Jobs page contains all **Verified Jobs** from the last 7 days.

Jobs are ranked by:

> **Application Priority Score**

The Top 10–20 opportunities should receive stronger visual emphasis.

---

# 46. Best Jobs Filters

The user should be able to filter by:

* Job role
* Seniority
* Location
* Work arrangement
* Minimum Application Priority Score
* Direct Email Application
* Freshness

Example:

```text
Role:
[AI Engineer ▼]

Seniority:
[Mid-level ▼]

Location:
[EMEA ▼]

Minimum Score:
[80]

Freshness:
[Last 24h]

☑ Direct Email Application
```

---

# 47. Suggested Job Card

Example:

```text
AI Engineer
Company X

🎯 Application Priority: 94/100

🟢 Verified
📧 Direct Email Application
🔥 Posted <24h

Remote — EMEA

Skills Match       94%
Experience Match   95%
Role Match         93%
Location           100%

Application email:
jobs@company.com

[Official Job Posting]
[Apply]
```

---

# 48. Job Card Details

A detailed job view should include:

## Basic Information

* Job title
* Company
* Location
* Work arrangement
* Seniority
* Posted date

## Verification

* Verification status
* Official source
* Verification date
* ATS if applicable

## Matching

* Overall Application Priority Score
* Skills score
* Experience score
* Role score
* Location score
* Freshness

## Resume Match

* Matched skills
* Missing skills
* Relevant experience
* Potential gaps

## Application

* Official job posting
* Direct application URL
* Application email when explicitly provided

## Discovery

* Platforms where the job was found

---

# 49. Resume Insights Page

The Resume Insights page should show:

## Strong Matches

Skills and experience that align well with the target jobs.

## Keyword Opportunities

Keywords that could be emphasized.

## Missing Keywords

Relevant terms that are frequently requested but absent from the resume.

## Recommendations

Specific actions the candidate can take to improve the discoverability of their existing experience.

---

# 50. Skills Market Page

The Skills Market page should show:

```text
Most Requested Skills
        ↓
Frequency
        ↓
Candidate Skill Status
        ↓
Learning Priority
```

Example:

| Skill      | Demand    | Candidate | Priority  |
| ---------- | --------- | --------- | --------- |
| Python     | Very High | Have      | Low       |
| AWS        | High      | Have      | Low       |
| RAG        | High      | Have      | Low       |
| Kubernetes | High      | Gap       | High      |
| LangGraph  | Med-High  | Gap       | Very High |
| Terraform  | Medium    | Gap       | Medium    |

---

# 51. Unverified Jobs Page

The Unverified Jobs page contains jobs that:

* Came from an approved discovery platform.
* Meet the freshness requirement.
* Appear relevant.
* Could not be officially confirmed.

Each should display:

```text
🟡 Unverified
```

The reason for the unverified status should be shown when possible.

Example:

```text
Official posting could not be confirmed.
```

---

# 52. Manual Search

The MVP uses **manual search only**.

The user initiates the process by clicking:

```text
Search Jobs
```

The system then executes:

```text
Discover
   ↓
Collect
   ↓
Normalize
   ↓
Filter
   ↓
Verify
   ↓
Deduplicate
   ↓
Match
   ↓
Rank
   ↓
Analyze
   ↓
Display
```

There is no continuous background job monitoring in the MVP.

---

# 53. Search Scope

The search should consider:

* Selected target roles
* Selected seniority
* Selected locations
* Selected work arrangements
* Candidate resume
* Job freshness

The system should prioritize relevant jobs rather than searching every possible software role.

---

# 54. Job Discovery Architecture

The discovery layer should be modular.

Example:

```text
Job Discovery Service
│
├── LinkedIn Discovery
├── Indeed Discovery
├── Glassdoor Discovery
├── Wellfound Discovery
├── Built In Discovery
├── Remote OK Discovery
├── We Work Remotely Discovery
├── Welcome to the Jungle Discovery
├── Dice Discovery
├── ZipRecruiter Discovery
│
├── Search Engine Discovery
└── Job API Sources
```

The architecture should make it possible to add or remove sources without changing the rest of the system.

---

# 55. Verification Architecture

Verification should be a separate service/module from discovery.

```text
Discovered Job
      |
      ↓
Find Company
      |
      ↓
Find Official Careers Page
      |
      ↓
Identify ATS
      |
      ↓
Search Exact Job
      |
      ↓
Compare Job Details
      |
      ↓
Check Status
      |
      +----------------+
      |                |
      ↓                ↓
  Verified         Unverified
```
