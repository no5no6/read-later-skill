# read-later-skill

![read-later-skill cover](./assets/read-later-skill-cover.png)

[English](./README.md) | [中文](./README.zh-CN.md)

A read-later skill repository that defines how article links shared in a chat or agent environment should be summarized, classified, deduplicated, and stored in MongoDB for later retrieval and follow-up questions.

This repository primarily contains skill instructions and reference documents, and is intended to serve as an Agent Skill / prompt-spec repository.

## Goals

- Accept article links from a chat or agent environment
- Extract structured metadata such as summary, tags, topic, and user notes
- Deduplicate records with `canonical_url` and `url_hash`
- Store records in the fixed collection `readlater.articles`
- Support later retrieval and follow-up questions using time range, topic, source site, and keyword filters

## Repository Structure

```text
.
├── SKILL.md
└── references/
    ├── mongodb-collection-schema.md
    ├── reply-templates.md
    └── smoke-test.md
```

### Files

- `SKILL.md`: the main skill definition, including scope, write flow, retrieval flow, response contract, and safety rules
- `references/mongodb-collection-schema.md`: the MongoDB document schema, recommended indexes, and query constraints
- `references/reply-templates.md`: reusable templates for save confirmations, question answering, and low-confidence fallback
- `references/smoke-test.md`: a five-step smoke test for validating the skill in a local chat window or any agent chat environment

## Core Workflow

### 1. Save an Article

When a user sends an article link, the skill should:

1. Normalize the URL and remove obvious tracking parameters
2. Build `url_hash` from the normalized URL
3. Deduplicate by `canonical_url` first, then by `url_hash`
4. Build a structured document
5. Insert a new record or update only the allowed fields in MongoDB
6. Return a short acknowledgment with the summary, tags, original URL, and record ID

Recommended fields:

- `title`
- `source`
- `original_url`
- `canonical_url`
- `summary`
- `key_points`
- `tags`
- `topic`
- `language`
- `user_note`
- `saved_at`
- `last_updated_at`

### 2. Retrieve and Answer

When the user asks about a previously saved article, the skill should:

1. Parse constraints such as time range, topic keywords, or source site
2. Query `readlater.articles` with those filters
3. Rank candidates by topic relevance and recency
4. Answer using the stored `summary`, `key_points`, and `evidence_snippets`
5. Ask one clarification question if confidence is too low

## MongoDB Conventions

This project uses:

- Database: `readlater`
- Collection: `articles`

See `references/mongodb-collection-schema.md` for the full schema. Key constraints include:

- All writes go to `readlater.articles`
- Dedup priority: `canonical_url` -> `url_hash`
- Use `saved_at` for time filtering
- Use `topic` + `tags` for topical retrieval
- Recommended write safety filter: `created_by = "read-later-skill"`

Recommended indexes:

```javascript
db.articles.createIndex({ url_hash: 1 }, { unique: true })
db.articles.createIndex({ canonical_url: 1 })
db.articles.createIndex({ saved_at: -1 })
db.articles.createIndex({ topic: 1, saved_at: -1 })
db.articles.createIndex({ tags: 1 })
db.articles.createIndex({ created_by: 1, saved_at: -1 })
```

## Response Conventions

The repository includes reusable reply templates for:

- save acknowledgments with the summary, tags, original URL, and record ID
- question answering with a conclusion, supporting points, the original URL, and the record ID
- low-confidence fallback with candidate articles and one follow-up question

See `references/reply-templates.md`.

## Smoke Test

You can validate the skill by following the 5-step flow in `references/smoke-test.md` from a local chat window or any agent chat environment:

1. Search recently saved articles
2. Fetch one record in detail
3. Create a test article record
4. Update the test record
5. Query the record back

If all five steps pass, the basic save-and-retrieval flow is working.

## Use Cases

- personal read-later collections
- article saving in conversational agents
- lightweight knowledge retention and retrieval on MongoDB
- structured context for later article Q&A

