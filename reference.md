# Reference — LinkedIn Trend Finder

Loaded by [SKILL.md](SKILL.md) on demand. Keep this file as the single source of truth for query patterns, taxonomies, and the extraction contract.

## 1. Content-pattern modifiers

Mix and match with the user's theme terms. Each modifier biases results toward a different hook style. Use **at least 4 different modifiers** across the query set.

| Modifier phrase | Biases toward |
| --- | --- |
| `"lessons learned"` | retrospective stories |
| `"what I wish I knew"` | mentor / hindsight posts |
| `"unpopular opinion"` | contrarian takes |
| `"here's what changed"` | news-reaction posts |
| `"we just shipped"` | launch / builder posts |
| `"stop doing"` | prescriptive lists |
| `"the real reason"` | insider framing |
| `"I just had"` | anecdotal openers |
| `"data shows"` / `"our data"` | stat-driven posts |
| `"hiring"` or `"we're hiring"` | recruiter-style posts |
| `"case study"` | long-form analysis |

## 2. Query templates

Substitute `<themes>` with 2–4 user theme terms. Keep queries short — DuckDuckGo/Google behind `WebSearch` handle 3–6 meaningful tokens best.

```
site:linkedin.com/posts/ <themes> "lessons learned" 2026
site:linkedin.com/posts/ <themes> "unpopular opinion"
site:linkedin.com/posts/ <themes> "what I wish I knew"
site:linkedin.com/posts/ <themes> "we just shipped" 2026
site:linkedin.com/posts/ <themes> "stop doing"
site:linkedin.com/posts/ <themes> "here's what changed"
site:linkedin.com/posts/ <themes> "case study"
site:linkedin.com/posts/ <themes> "data shows"
```

For a **freshness_days ≤ 14** window, append `"this week"` or `"last week"` instead of the year.
For **freshness_days ≤ 45**, append `"last month"`.

## 3. Hook-pattern taxonomy

Classify each post's opening line into exactly one:

| `hook_pattern` | Signal in the opening |
| --- | --- |
| `question` | Opens with a direct question to the reader |
| `bold-claim` | Declarative, opinionated, no hedging ("X is dead.") |
| `story` | First-person narrative opener ("Last week I...") |
| `list` | Numbered or bulleted promise ("5 things...") |
| `stat` | Leads with a number or data point |
| `contrarian` | Explicitly pushes against consensus ("Everyone says X. They're wrong.") |
| `how-to` | Tutorial framing ("How to X in Y steps") |

If ambiguous, pick the pattern that best matches the **first two sentences**, not the body.

## 4. Post-format classifier

| `format` | How to recognize from the fetched page |
| --- | --- |
| `text` | No media attached |
| `image` | One or more `<img>` attachments in the post body (not author avatar) |
| `video` | Video player element or `og:video` meta |
| `carousel` | LinkedIn document/slide carousel (look for "document" in OG type or slide indicators) |
| `poll` | Poll markup or "voted" / "X votes" in visible text |
| `article` | URL path contains `/pulse/` instead of `/posts/` |

## 5. Extraction prompt (pass verbatim to `WebFetch`)

When calling `WebFetch(url, prompt)`, use this as the prompt:

> Extract the following from this public LinkedIn post page. Return a JSON object. Use `null` for any field not visible on the page — do not guess.
>
> - `author.name`: the post author's full name
> - `author.title`: their headline/role text shown under the name
> - `author.profile_url`: the `/in/...` profile URL
> - `author.followers`: follower count if shown, as an integer
> - `headline`: the first line of the post body (the hook)
> - `content`: the full post text, preserving line breaks
> - `posted_at`: the visible post date (ISO `YYYY-MM-DD` if possible, otherwise the relative string like "2w")
> - `engagement.reactions`: total reaction count as integer
> - `engagement.comments`: comment count as integer
> - `engagement.shares`: repost/share count as integer
> - `media`: array of `{type, url}` for attached images/videos/documents (exclude the author avatar)
> - `format`: one of `text | image | video | carousel | poll | article`
>
> If the page is a login wall or the content is hidden, return `{"blocked": true}`.

## 6. Output JSON schema (authoritative)

```json
{
  "url": "https://www.linkedin.com/posts/<slug>",
  "author": {
    "name": "string",
    "title": "string|null",
    "profile_url": "string|null",
    "followers": "integer|null"
  },
  "headline": "string",
  "content": "string",
  "media": [{"type": "image|video|document", "url": "string"}],
  "engagement": {
    "reactions": "integer|null",
    "comments": "integer|null",
    "shares": "integer|null"
  },
  "posted_at": "string|null",
  "format": "text|image|video|carousel|poll|article",
  "hook_pattern": "question|bold-claim|story|list|stat|contrarian|how-to",
  "themes": ["string"],
  "relevance_score": 0.0,
  "relevance_reason": "string",
  "source_query": "string"
}
```

`posts.json` is a JSON array of these objects, sorted by composite rank (descending).

## 7. `posts.md` layout

```markdown
# LinkedIn Trend Finder — <brief one-line>

Generated <ISO timestamp> · <N> posts · freshness <D> days

| # | Author | Hook | Format | Reactions | Comments | URL |
| - | ------ | ---- | ------ | --------- | -------- | --- |
| 1 | ... | ... | ... | ... | ... | [link](...) |

## Patterns observed

- **Dominant hook styles:** story (6), bold-claim (3), list (2), ...
- **Dominant formats:** image (7), text (4), video (1)
- **Recurring themes:** ...
- **Notable authors (≥2 posts in set):** ...
```

## 8. `brief.md` layout

```markdown
# Brief

Raw input: <verbatim user brief>

Parsed:
- Themes: ...
- Audience: ...
- Tone/format hints: ...
- Freshness window: <D> days
- Target count: <N>

## Queries executed
1. `site:linkedin.com/posts/ ... "lessons learned" 2026`
2. ...

## Dropped URLs (login-walled or off-topic)
- <url> — reason
```
