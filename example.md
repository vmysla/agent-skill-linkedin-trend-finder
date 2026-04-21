# Example run

An end-to-end walkthrough of a real invocation, including the exact prompt, the files the skill produces, and the inline response. Use this to calibrate what to expect before running your own brief.

## 1. Invocation

```
/linkedin-trend-finder The future of product management in an agentic AI world.
```

The brief is intentionally short — no audience, tone, or freshness hints. The skill fills in defaults rather than asking clarifying questions, because the theme (`product management` + `agentic AI`) is clear from the sentence.

## 2. Parsed brief

The skill extracts and records the following (see [brief.md](#5-briefmd) below for the full version):

| Field | Value |
| --- | --- |
| Themes | product management, agentic AI, AI agents, future of PM, AI PM role |
| Audience | product managers, PM leaders, AI/product builders |
| Tone/format hints | not specified — accept a mix of styles |
| Freshness window | 90 days (default) |
| Target count | 12 (default) |

## 3. Queries executed

Ten `WebSearch` calls were fired in parallel, rotating across content-pattern modifiers from [reference.md](reference.md):

```
site:linkedin.com/posts/ product management agentic AI "lessons learned" 2026
site:linkedin.com/posts/ product manager agents "unpopular opinion"
site:linkedin.com/posts/ product management AI agents "what I wish I knew"
site:linkedin.com/posts/ future of product management agentic 2026
site:linkedin.com/posts/ PM role AI agents "stop doing"
site:linkedin.com/posts/ product management agentic "here's what changed"
site:linkedin.com/posts/ agentic product manager "case study" 2026
site:linkedin.com/posts/ AI agents product management "the real reason"
site:linkedin.com/posts/ "AI PM" agents workflow 2026 "here's what"
site:linkedin.com/posts/ product management AI agents "we just shipped" 2026
```

## 4. Output file tree

```
output/
└── 2026-04-21-1103/
    ├── posts.json   # 12 posts, full schema, sorted by composite rank
    ├── posts.md    # ranked table + patterns observed
    └── brief.md    # parsed brief, queries executed, dropped URLs
```

The folder name is `<YYYY-MM-DD-HHMM>` in the current working directory. Every run creates a new folder, so old runs are preserved.

> The `output/` folder is gitignored by default — results are intended as ephemeral artifacts handed to downstream agents, not committed.

## 5. `brief.md`

Reproducibility record. Contains the raw input, the parsed fields, every query fired, and any URLs dropped during filtering with the reason. Drop categories include: off-topic, login-walled, too old, or thin snippet with no extractable body.

Excerpt:

```markdown
Raw input: the future of product management in agentic world.

Parsed:
- Themes: product management, agentic AI, AI agents, future of PM, AI PM role
- Audience: product managers, PM leaders, AI/product builders considering the
  PM discipline's next decade
- Freshness window: 90 days (default)
- Target count: 12 (default)

## Dropped URLs (login-walled, off-topic, or too old to enrich the set)
- https://www.linkedin.com/posts/johnshehata_... — off-topic (LinkedIn content
  automation advice, not PM/agentic)
- https://www.linkedin.com/posts/dhavalbhatt_... — 2023 program announcement,
  superseded by newer AI PM posts in this set
```

## 6. `posts.md`

Ranked Markdown table plus a `## Patterns observed` block. Excerpt:

```markdown
| # | Author | Hook | Format | Reactions | Comments | URL |
| - | ------ | ---- | ------ | --------- | -------- | --- |
| 1 | Kevin Thomas (IBM) | AI Product Management in 2026 is much more than
  shipping Agents and RAGs. | text | 63 | 14 | [link](...) |
| 2 | LinkedIn News Europe | We're excited to release our annual list of Jobs
  on the Rise in Europe. | article | 1093 | 110 | [link](...) |
| 3 | Kemi O. | Unpopular opinion: AI agents aren't ready for every business. |
  text | 6 | — | [link](...) |
...

## Patterns observed

- Dominant hook styles: contrarian (5), bold-claim (3), list (1), how-to (1),
  question (1), story (1), stat (1).
- Dominant formats: text (8), article (4) — no video/poll/carousel.
- Recurring themes: workflow redesign, agent governance, memory/context as
  moat, PM role redefinition, agent readiness skepticism.
- Freshness caveat: only 2 of 12 posts fall within the 90-day window — the
  agentic-PM conversation is newer than LinkedIn's public crawl.
```

`posts.md` is the file to read first when you want a human-scannable overview.

## 7. `posts.json`

The machine-readable deliverable. A JSON array of 12 post objects, sorted by composite rank, matching the schema in [reference.md §6](reference.md). Downstream agents (reposter, commenter, lookalike author) consume this directly.

One entry, abridged:

```json
{
  "url": "https://www.linkedin.com/posts/kevinthomasjoseph_...",
  "author": {
    "name": "Kevin Thomas",
    "title": "IBM",
    "profile_url": "https://in.linkedin.com/in/kevinthomasjoseph",
    "followers": 19000
  },
  "headline": "AI Product Management in 2026 is much more than shipping Agents and RAGs.",
  "content": "AI Product Management in 2026 is much more than shipping Agents and RAGs. ...",
  "media": [],
  "engagement": { "reactions": 63, "comments": 14, "shares": null },
  "posted_at": "1mo",
  "format": "text",
  "hook_pattern": "bold-claim",
  "themes": ["AI product management", "agentic AI", "PM fundamentals"],
  "relevance_score": 0.95,
  "relevance_reason": "Directly answers 'future of PM in an agentic world' — five concrete shifts for AI PMs in 2026 including redesigning workflows, trust design, governance.",
  "source_query": "site:linkedin.com/posts/ future of product management agentic 2026"
}
```

Every field is populated from the public page or set to `null` — never fabricated.

## 8. Inline response

Finally, the skill prints a short summary to the chat. For this run:

```
Output: output/2026-04-21-1103/ — posts.json, posts.md, brief.md.

Top 3
1. Kevin Thomas (IBM) — "AI Product Management in 2026 is much more than shipping
   Agents and RAGs" — <url>
2. LinkedIn News Europe — "Jobs on the Rise 2026: work is moving to
   agent-orchestrated networks" — <url>
3. Kemi O. — "Unpopular opinion: AI agents aren't ready for every business" — <url>

Pattern: contrarian hooks + numbered implication lists dominate. Caveat: only 2
of 12 posts fall inside the 90-day window — topic is newer than the public index.
```

The inline response is deliberately short. The files are the deliverable.

## 9. How to consume the output

- **Read first:** open `posts.md` for a quick human overview and the patterns block.
- **Feed to an agent:** point a downstream skill at `posts.json` — the schema is stable and documented in [reference.md §6](reference.md).
- **Audit the run:** `brief.md` shows exactly which queries fired and which URLs were dropped, so you can rerun with a tweaked brief if the mix isn't right.

## 10. Tips for writing better briefs

- **Add a freshness hint** when the topic moves fast: `... last 30 days` drops `freshness_days` to 30 and switches recency tokens to `"this week"` / `"last month"`.
- **Name an audience** when your theme is broad: `... for early-stage founders` narrows relevance judgments sharply.
- **Ask for a different count** when you need a lookalike corpus: `... give me 25 posts`.
- **Name a tone** when the downstream use is writing: `... tactical, story-driven` biases the hook-pattern selection.
