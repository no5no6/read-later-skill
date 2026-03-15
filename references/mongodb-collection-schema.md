# MongoDB Collection Schema

Use one fixed collection: `readlater.articles`.

## Document shape

```json
{
  "_id": "ObjectId",
  "title": "string",
  "canonical_url": "string",
  "url_hash": "string",
  "original_url": "string",
  "source": "string",
  "saved_at": "date",
  "last_updated_at": "date",
  "status": "Unread|Reading|Done|Archived",
  "topic": "AI|Engineering|Product|Business|Design|Data|Other",
  "tags": ["string"],
  "language": "zh|en|other",
  "user_note": "string",
  "summary": "string",
  "key_points": ["string"],
  "evidence_snippets": ["string"],
  "created_by": "read-later-skill"
}
```

## Required indexes

```javascript
db.articles.createIndex({ url_hash: 1 }, { unique: true })
db.articles.createIndex({ canonical_url: 1 })
db.articles.createIndex({ saved_at: -1 })
db.articles.createIndex({ topic: 1, saved_at: -1 })
db.articles.createIndex({ tags: 1 })
db.articles.createIndex({ created_by: 1, saved_at: -1 })
```

## Query hints

- Dedup: `canonical_url` then `url_hash`.
- Time filter: `saved_at`.
- Topic retrieval: `topic` plus `tags`.
- Safety filter for writes: `created_by = "read-later-skill"`.
