# LinkedIn Trend Finder

A Claude Code skill that discovers trending public LinkedIn posts for a given company, theme, or audience, and returns a ranked structured list that downstream agents can use to repost, comment on, or author lookalike content.

## How it works

- **Discovery:** built-in `WebSearch` with `site:linkedin.com/posts/` scoping and a rotating set of content-pattern modifiers (e.g. `"lessons learned"`, `"unpopular opinion"`).
- **Extraction:** built-in `WebFetch` per finalist post, pulling author, hook, full text, engagement, media, and date from the public page.
- **Ranking:** relevance × log(engagement) × recency decay.

No API keys, no external MCP servers, no scraping dependencies. Free and works out of the box inside Claude Code.

## Install

Symlink (or copy) this repo into your Claude Code skills directory:

```sh
ln -s "$PWD" ~/.claude/skills/linkedin-trend-finder
```

Restart Claude Code; the skill will appear in the skills list.

## Use

Pass a free-form brief — anything from a one-liner to a full company description:

```
/linkedin-trend-finder AI strategy consulting for enterprise leaders — post ideas on ROI and implementation pitfalls, last 30 days
```

The skill parses themes, audience, freshness window, and target count from the brief. If the brief is too thin to identify a theme, it asks one clarifying question.

## Output

Written to `output/<YYYY-MM-DD-HHMM>/` in the current working directory:

- **`posts.json`** — ranked array of posts, full schema (see [reference.md](reference.md)).
- **`posts.md`** — ranked Markdown table plus a "Patterns observed" summary.
- **`brief.md`** — parsed brief and the exact queries executed, for reproducibility.

The inline response gives the output path and the top 3 posts.

See [example.md](example.md) for an end-to-end walkthrough — a real brief, the parsed fields, the queries fired, and excerpts of every output file.

## Files

- [SKILL.md](SKILL.md) — skill entry point and workflow.
- [reference.md](reference.md) — query templates, hook taxonomy, extraction prompt, output schema.
- [example.md](example.md) — worked example of a full run, with output snippets and tips for writing better briefs.
