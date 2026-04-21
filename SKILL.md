---
name: linkedin-trend-finder
description: >
  Discover trending, high-performing public LinkedIn posts for a given company, theme,
  or audience, and return a ranked structured list (JSON + Markdown) that downstream
  agents can use to repost, comment, or author lookalike content.
when_to_use: >
  Use when the user asks for LinkedIn post ideas, wants to see what's trending on LinkedIn
  in a niche, needs inspiration for a company page, is researching what a target audience
  engages with on LinkedIn, or wants raw material for a reposting/commenting agent. Do NOT
  use for LinkedIn profile lookups, job listings, or messaging automation.
user-invocable: true
argument-hint: "[free-form brief: company, themes, audience, goals]"
---

# LinkedIn Trend Finder

Given a free-form brief, find ~12 trending public LinkedIn posts that match, extract full structured data for each, and write results to disk for downstream agents.

## Inputs

The brief is whatever the user passes after the skill name — company description, themes, audience, goals, tone, freshness window, or any mix. Parse it yourself.

Fields to extract from the brief (with defaults):

- `themes`: core topics/keywords — required
- `audience`: who the posts should resonate with — optional
- `tone_format_hints`: e.g. "tactical", "story-driven", "data-heavy" — optional
- `freshness_days`: recency window — default **90**
- `num_results`: target count — default **12**

If the brief is under ~5 words AND you cannot identify a theme, ask **one** clarifying question via `AskUserQuestion` with 2–4 plausible topic options. Otherwise proceed — do not interrogate.

## Workflow

### 1. Plan
Load [reference.md](reference.md) for query templates, hook taxonomy, the extraction prompt, and the output schema. Write the parsed brief and the query plan to memory (you'll persist it in step 7).

### 2. Generate queries
Build **6–10** `WebSearch` queries by combining:
- the user's theme terms (2–4 words each),
- one or more **content-pattern modifiers** from `reference.md` (e.g. `"lessons learned"`, `"unpopular opinion"`, `"here's what I learned"`),
- the scope `site:linkedin.com/posts/`,
- a recency hint matching `freshness_days` (e.g. `2026`, `"this week"`, `"last month"`).

Vary modifiers across queries so you sample different hook styles.

### 3. Discover
Run all queries **in parallel** (single message, multiple `WebSearch` tool calls). For each result, record `url`, `title`, `snippet`, `source_query`. Dedupe by canonical LinkedIn post URL (strip query strings, normalize `/posts/` slug).

### 4. Pre-rank on snippets
Using only title + snippet, judge relevance to the brief. Keep the top `2 × num_results` as finalists. Discard anything that is clearly a profile page, job listing, or company page rather than a post.

### 5. Extract
`WebFetch` each finalist **in parallel** (batched in groups of ~5 tool calls per message to stay responsive) using the extraction prompt from `reference.md`. Capture every field in the output schema. When a field isn't visible on the public page, set it to `null` — never fabricate.

### 6. Score & classify
For each post:
- `relevance_score` (0–1): judgment vs. the brief.
- `hook_pattern`: pick from the taxonomy in `reference.md`.
- `themes`: 1–3 tags drawn from the brief and post content.
- Composite rank: `relevance_score × log10(max(engagement_total, 10)) × recency_decay`, where `recency_decay = 0.5 ^ (age_days / 30)` when `posted_at` is known, else `1.0`.

Sort descending; keep the top `num_results`.

### 7. Write outputs
Create `output/<YYYY-MM-DD-HHMM>/` in the current working directory and write three files:

- **`posts.json`** — array of posts matching the schema in `reference.md`.
- **`posts.md`** — ranked table (`#`, author, hook preview, format, engagement, URL) followed by a `## Patterns observed` section listing dominant hook styles, formats, and themes across the set.
- **`brief.md`** — the parsed brief (themes, audience, freshness, count) and the exact query list executed, for reproducibility.

### 8. Report
Respond with:
- the output folder path,
- a 3-line summary of the top 3 posts (author — hook — URL),
- one sentence on patterns observed.

Keep the inline response short; the files are the deliverable.

## Guardrails

- Only fetch **public** `linkedin.com/posts/` URLs. If a URL redirects to a login wall, drop it and note it in `brief.md` under "dropped".
- Never fabricate author names, follower counts, or engagement numbers. `null` when unknown.
- Do not attempt authenticated scraping, cookie injection, or API keys. WebSearch + WebFetch only.
- If fewer than 5 candidates survive filtering, say so explicitly in the report — do not pad with irrelevant posts.
