# RhythmDB

an experimental embedded database for [MECOMP](https://github.com/AnthonyMichaelTDM/mecomp)

## feature-list

- store key-value data
  - [ ] fixed-size data
  - [ ] variable-length data
  - [ ] arbitrary serializable structs (apache arrow?)
- [ ] crud operations
- [ ] table scans (iterating over a table) and direct accessing (efficiently finding a specific key in the table)
- [ ] atomic transactions (ACID compliant?)
- support for the following indexes:
  - [ ] standard index: a single column value (or tuple of values) (key) maps to potentially many record ids (values)
  - [ ] unique index: a single column value (or tuple of values) (key) maps to a single record id (value)
  - [ ] vector index: an index used for efficient k-nearest-neighbors searches
    - preferably able to persist (or at least spill) to disk
    - should eventually support multiple kinds of indexes, but only supporting one is fine for version 1.0
    - should have space complexity < O(n^2)
  - [ ] full text search: map text chunks to the record id's that match them
    - [ ] support an array of tokenizers and filters: <https://surrealdb.com/docs/surrealdb/models/full-text-search>
    - [ ] must be able to score results so that query results can be ranked by score, like: `SELECT *, search::score(0) as relevance FROM documents WHERE text @0@ "lorem ipsum" ORDER BY relevance`
- [ ] support computed fields
  - if I do go down the route of having a query language, this might be hard (might have to embed a simple scripting language). But if I go for the simpler option of having all queries be built up as structs, then a computed field can just be a rust closure that gets run to compute the field when needed.
  - computed fields cannot be indexed (duh) and should not be used as part of the query unless I can somehow guarantee that the function is pure and idempotent.
  - might decide to place the burden on users to compute their own computed fields

## Concessions

Making a database from-scratch is a very difficult endeavour, I'm okay with using
existing libraries for the underlying storage engine, query planner, etc.
As long as I can still keep the number of dependencies low (ideally > 200-300).

- currently favoring [redb](https://github.com/cberner/redb) as the storage engine.

Since I'm not expecting other people to use this, I'm okay with not having a query planner,
but effort should be taken to make common operations as easy as possible for users
w/o needing to mess around with lifetime shenanigans or similar.

Schemas do not need to be mutable, it's okay for the schema to be defined at compile-time if that enabled better optimizations.

## Requirements

Ideally, most if not all of the index types should be stored as tables of their own instead of operating purely in-memory.

The (time) performance of this DB *must* match or exceed that of surrealdb for the use-cases of [mecomp](https://github.com/AnthonyMichaelTDM/mecomp).

The DB ***must*** have fewer dependencies then surrealDB.

## Development

### Git Hooks

RhythmDB uses several git hooks to ensure code quality, these are stored in the `.githooks` directory, to install these hooks, run:

```sh
git config core.hooksPath .githooks
```
