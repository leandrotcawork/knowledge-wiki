# Knowledge Wiki Schema

## Article Quality Rules

1. Every article MUST have YAML frontmatter: domain, confidence, sources, last_updated.
2. Confidence levels: high (3+ trusted sources), medium (1-2 sources or non-trusted), low (single source or outdated).
3. Write like a senior engineer documenting implementation knowledge. Dense, practical, zero fluff.
4. Sections adapt to the topic — no fixed template. Quality is constant, structure is flexible.
5. Always include "## See Also" with [[backlinks]] and "## Sources" with raw/ paths.
6. Raw sources are immutable — never edit files in raw/.

## When to Create vs Update

- CREATE when a distinct concept/entity/tool doesn't have its own page.
- UPDATE when new sources refine or extend existing knowledge.
- FLAG contradictions rather than silently overwriting.

## Naming Conventions

- File names: lowercase, hyphenated. Ex: `oauth2.md`, `session-management.md`
- Domain paths: `domains/<area>/<sub-area>/<topic>.md`
- Each domain folder has `_index.md` with overview and links.
- Entities: `entities/<tool-name>.md`
- Concepts: `concepts/<concept-name>.md`

## Tech Stack

When articles include code examples, prefer these languages and frameworks:
- Python (FastAPI, SQLAlchemy, Pydantic)
- Go (standard library, gin, sqlx)

If raw sources contain examples in other languages, translate the patterns
to the preferred stack. Keep the original only if the topic is specifically
about that language/framework.

## Log Format

`## [YYYY-MM-DD HH:MM] <operation> | <title>` followed by 1-3 lines.
Operations: ingest, update, audit, proactive.