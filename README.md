# read-later-skill

[English](./README.md) | [中文](./README.zh-CN.md)

A channel-agnostic skill for saving articles for later reading. It defines how links shared in a chat or agent environment should be summarized, classified, deduplicated, stored in MongoDB, and later retrieved with evidence-backed answers.

This repository mainly contains skill instructions and reference documents. It is meant to serve as an Agent Skill / prompt-spec repository rather than a full runtime application.

## Goals

- Accept article links from a chat or agent environment
- Extract structured metadata such as summary, tags, topic, and user notes
- Deduplicate records with `canonical_url` and `url_hash`
- Store records in the fixed collection `readlater.articles`
- Support later retrieval by time range, topic, source, and keywords

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

- `SKILL.md`: main skill definition, including scope, write flow, retrieval flow, response contract, and safety rules
- `references/mongodb-collection-schema.md`: MongoDB document shape, recommended indexes, and query constraints
- `references/reply-templates.md`: response templates for save confirmation, question answering, and low-confidence fallback
- `references/smoke-test.md`: a 5-step smoke test for validating the skill in a local chat window or any agent chat environment

## Core Workflow

### 1. Save an Article

When a user sends an article link, the skill should:

1. Normalize the URL and remove obvious tracking parameters
2. Build `url_hash` from the normalized URL
3. Deduplicate by `canonical_url` first, then by `url_hash`
4. Build a structured document
5. Insert or update allowed fields in MongoDB
6. Return a short acknowledgment with summary, tags, original URL, and record ID

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
4. Answer from stored `summary`, `key_points`, and `evidence_snippets`
5. Ask one clarification question if confidence is too low

## MongoDB Conventions

This project uses:

- Database: `readlater`
- Collection: `articles`

See `references/mongodb-collection-schema.md` for the full schema. Key constraints:

- All writes go to `readlater.articles`
- Dedup priority: `canonical_url` -> `url_hash`
- Use `saved_at` for time filtering
- Use `topic` + `tags` for topic retrieval
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

- save acknowledgment with summary, tags, original URL, and record ID
- question answering with conclusion, supporting points, original URL, and record ID
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

- personal read-later collection
- article saving in conversational agents
- lightweight knowledge retention and retrieval on MongoDB
- structured context for later article Q&A

## Current Scope

This repository is currently documentation-first. Good next additions would be:

- channel integration examples such as Telegram, Web Chat, or CLI
- URL normalization implementation
- summary and tagging strategy
- MongoDB MCP usage examples
- automated checks or scripted validation
