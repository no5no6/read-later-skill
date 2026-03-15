---
name: read-later-skill
description: Use when the user sends article links in a chat or agent environment for save-for-later. Summarize and classify the article, store it in MongoDB via MongoDB MCP, and answer later questions with evidence and the source link.
---

# Read Later Skill

## When to use

Use this skill when the user wants to:

- save links for later reading,
- get concise summaries and tags,
- ask follow-up questions like "what did that article say?",
- retrieve original links quickly.

## Preconditions

- MongoDB MCP is connected and write-enabled.
- Target database and collection are fixed:
- Database: `readlater`
- Collection: `articles`
- The collection follows `references/mongodb-collection-schema.md`.

## Core workflow

1. Receive link + optional user note from the current chat or agent environment.
2. Normalize URL (remove obvious tracking params).
3. Build `url_hash` from normalized URL.
4. Deduplicate:
- First by `canonical_url`.
- Fallback by `url_hash`.
5. Build structured document:
- title,
- source site,
- user note,
- short summary,
- key points,
- tags,
- topic,
- language,
- saved time.
6. Write to MongoDB:
- Insert if not found.
- Else update only whitelisted fields.
7. Return acknowledgment:
- one-sentence summary,
- tags,
- record id,
- original URL.

## Retrieval workflow

When user asks questions (for example "last week the xxx article said what?"):

1. Parse constraints:
- time range (for "last week", use the previous Monday-Sunday in user locale),
- topic keywords,
- optional source/site.
2. Query MongoDB documents with filters.
3. Rank candidates by topic/tag match and recency.
4. Answer with evidence-first format:
- short answer,
- 2-4 bullet key points,
- original article URL,
- record id.
5. If confidence is low, ask one clarification question before final answer.

## Response contract

Always include:

- `Answer`: direct response in plain language.
- `Why`: evidence from stored summary/key points.
- `Links`: original URL first, record id second.

Never invent article details. If record data is insufficient, say so explicitly.

## Write policy

- Keep summary to 120-220 Chinese characters by default.
- Keep key points to 3-5 bullets.
- Tag count: 3-6.
- Prefer stable topic taxonomy:
- `AI`
- `Engineering`
- `Product`
- `Business`
- `Design`
- `Data`

## Read policy

- For "what did it say", prefer stored `Key Points` and `Evidence Snippets`.
- For "exact wording", return best available snippet and source link.
- For "which one was that article", return top 3 candidates with links.

## Safety rules

- Never write outside `readlater.articles`.
- Never upsert without dedupe check.
- Only update these fields:
- `title`
- `summary`
- `key_points`
- `tags`
- `topic`
- `user_note`
- `last_updated_at`
- If MongoDB MCP is unavailable, return an explicit error and ask for MCP fix. Do not fallback to local memory files.

## References

- Collection schema: `references/mongodb-collection-schema.md`
- Answer templates: `references/reply-templates.md`
- Smoke test: `references/smoke-test.md`
