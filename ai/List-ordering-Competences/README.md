# competency-taxonomy

Turns a flat, unordered list of competency labels into a two-level taxonomy with stable integer indexes, output as YAML.

The input is the kind of list that accumulates in a database over time: professional areas, programming languages, frameworks and tools all mixed together at different levels of granularity, in whatever order they were added. The output groups them under headings a non-technical reader can scan, and assigns each entry an integer that never changes as the list grows.

## What it does

- **Groups by parent.** Laravel and Symfony land under PHP, PostgreSQL and MySQL under a databases heading.
- **Flattens to exactly two levels.** Where the natural taxonomy runs deeper, the most specific parent wins and the broader one is dropped.
- **Invents headings** where the input has no suitable parent, in the output language.
- **Assigns readable integers** — `7005` is category 7, item 5 — that survive additions and deletions.
- **Reports its judgement calls** so ambiguous placements can be overruled rather than silently accepted.

## Input

| Input | Required | Default |
|---|---|---|
| Flat list of labels | Yes | — |
| Output language | No | English |
| Existing taxonomy file | On re-runs only | None; a fresh index sequence is generated |
| Max top-level categories | No | 10 |
| Max items per category | No | 999 |

The list can be pasted as plain text, comma-separated, or as raw HTML — if it's HTML, point at the element holding the labels (`span.truncate`, for instance) and they'll be extracted for you. Casing and duplicates don't matter.

Output is English unless you ask for another language, and the language applies to generated text only: invented headings, the "plain" variants, and the fallback item. Technology names keep their conventional spelling regardless, so PostgreSQL stays PostgreSQL whatever language you pick.

**Example input** (10 labels):

```
mysql, postgreSQL, PHP, Laravel, Symfony, Javascript, ReactJS, backend, frontend, QA
```

Output language left at the default.

## Output

YAML. A flow mapping of next-free subindexes, then one flat list of quoted `"<integer>:<label>"` strings with a blank line between categories.

```yaml
next_subindex: { 1: 4, 2: 3, 3: 5, 4: 4, 5: 5 }

competencies:
  - "1:Professional areas"
  - "1001:Backend"
  - "1002:Frontend"
  - "1003:Other… see CV"

  - "2:Roles and methods"
  - "2001:QA"
  - "2002:Other… see CV"

  - "3:PHP"
  - "3001:Plain PHP"
  - "3002:Laravel"
  - "3003:Symfony"
  - "3004:Other… see CV"

  - "4:JavaScript"
  - "4001:Plain JavaScript"
  - "4002:ReactJS"
  - "4003:Other… see CV"

  - "5:Databases"
  - "5001:MySQL"
  - "5002:PostgreSQL"
  - "5003:Other… see CV"
```

Three things in that output aren't in the input, and they're deliberate:

- **`Other… see CV`** closes every category, so a person whose skill isn't listed still has something to tick.
- **`Plain PHP` and `Plain JavaScript`** exist because PHP and JavaScript became headings — without them, someone who knows the language but no framework has nothing to select.
- **`Professional areas`, `Roles and methods`, `Databases`** are invented headings. `Backend` and `Frontend` are broad areas that were labels in the input, so they're gathered into one category rather than becoming headings that would crowd out the languages.

The two structural keys — `next_subindex` and `competencies` — are translated along with everything else if you choose a non-English output language. If your parser expects fixed key names, say so and they'll stay English regardless.

After the YAML comes a short prose list of judgement calls: invented headings, ambiguous parents, assumed spelling fixes, forced merges, and every newly issued integer.

## Index scheme

Category integers run 1–10. Item integers are `category * 1000 + subindex`. Decode with `intdiv($i, 1000)` and `$i % 1000`; any value ≤ 10 is a category. Split labels on the **first** colon only, so a label containing a colon can't corrupt the index.

Indexes are never reused, never renumbered, and never reassigned on rename. New items append at the next free subindex in their band, which is why the `next_subindex` high-water mark is stored — deleting the highest item in a band otherwise loses the next free number.

Two consequences worth knowing before you build against this:

- **File order is not index order.** Add items to a category and its `Other… see CV` keeps its low subindex while staying last in the file. Render from file order, not from sorting the integers.
- **Moving an item between categories changes its integer.** The index encodes position, so re-parenting is a delete plus an add. Anything storing the old value breaks.

## Capacity

Ten categories of 999 items is nearly 10,000 slots, but the readable limit is roughly a tenth of that: about 15 items per category and 100–150 in total. Past that, headings have to grow broad enough to swallow twenty-plus items and stop describing their contents.

Above roughly 150 labels the skill stops and asks before generating. The usual fix is raising the category cap to 16, which still packs into two bytes and changes nothing else.

Watch for one specific squeeze: every language that needs its own category consumes a top-level slot. Six languages leaves four slots for databases, servers, operating systems, styling, tooling and everything else.
