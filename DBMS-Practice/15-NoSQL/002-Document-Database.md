# Document Database

## Scenario
A CMS stores articles where each article type (blog post, video, podcast episode) has a meaningfully different set of fields (blog posts have `body_html`; podcasts have `audio_url` and `duration_seconds`) - forcing them all into one rigid relational schema means dozens of nullable, type-specific columns.

## Diagram
```
Document store (e.g. MongoDB):
{ "_id": 1, "type": "blog", "title": "...", "body_html": "..." }
{ "_id": 2, "type": "podcast", "title": "...", "audio_url": "...", "duration_seconds": 1800 }
   -- each document's shape can differ, no shared rigid schema required
```

## Problem
Contrast the relational approach (one wide table with many nullable columns, or several joined subtype tables) with a document approach for this content model.

## Solution
Relational option A (wide table): `articles(id, type, title, body_html NULL, audio_url NULL, duration_seconds NULL, ...)` - works, but grows messier and more nullable-column-heavy as content types multiply, and adding a new content type means a schema migration.

Relational option B (subtype tables + joins): normalized but requires a join per content type to reconstruct a full article, and adding a type still means a new table + migration.

Document approach: each article is stored as a self-contained document matching only the fields relevant to its type; adding a new content type ("newsletter") requires no schema migration at all - the application just starts writing documents with a new shape.

## Takeaway
Document databases fit naturally when your entities are heterogeneous (different "shapes" of the same conceptual thing) and evolve frequently - the cost is weaker cross-document consistency/join support compared to a relational database, a real tradeoff, not a free lunch.
