Flat list of competency labels (broad areas, technologies, tools/frameworks), unordered, mixed casing. Return as YAML, two levels only. Output language English unless told otherwise.

Capacity — check before generating:
1. Count the input labels first. Hard limits: 10 categories, 999 items per category. Practical limits: ~15 items per category, ~100-150 total. Under 100 proceed; 100-150 proceed but flag that categories are running fat; above 150 stop and ask first.
2. When stopping, offer in this order: raise the category cap to 16 (still two bytes, nothing else changes); accept broader headings; split into several taxonomies; allow depth 3 (breaks the index scheme — mention last).
3. Check specifically whether the input holds many languages. Each language needing its own category consumes a top-level slot; six leaves four for everything else.

Structure:
4. Depth 2 max: category + items. For chains (Backend>Databases>PostgreSQL) keep the most specific parent as heading, drop broader.
5. Max 10 categories. Merge related small categories to fit; report every merge. Merges compound — four of them produce headings that describe nothing.
6. Every label appears exactly once. Dedupe case-insensitively, normalise casing, drop nothing. Unplaceable -> "Unplaced".
7. Invent category names where no parent exists. Plain, descriptive of contents, understandable to a non-technical reader.
8. Broad professional areas that appear as labels -> all under "Professional areas".
9. A technology that is both heading and standalone skill (PHP, JavaScript) gets "Plain <name>" as its first item. Applies also when the heading was invented and the language never appeared as a label.
10. "Other… see CV" is the last item of every category, including single-item ones. Never a category itself.
11. Order: categories broad-to-specific; items by common usage, plain variants first, "Other… see CV" last.

Indexing:
12. Every entry is a single quoted string "<integer>:<label>". Integers only.
13. Category integer = 1-10. Item integer = category * 1000 + subindex, subindex 1-999.
14. New items take the next free subindex in their band. Never renumber, never reuse a deleted index, never reassign on rename.
15. Re-parenting changes the integer; treat as delete + add and flag it.
16. Integers need not run consecutively, and file order is not index order. Do not tidy. Warn once that consuming code must render from file order, not by sorting integers.
17. Each "Other… see CV" is a distinct item with its own integer.
18. On re-run with an existing file as input: preserve every existing integer, assign new ones only to genuinely new labels.

Output:
19. YAML only. `next_subindex` flow mapping (category -> next free subindex), then `competencies:` as a single flat list of quoted strings, blank line between categories. No nesting, no comments.
20. Quote every entry — unquoted, "1001:Backend" parses as a mapping.

After the YAML, list judgement calls: merges forced by the cap, ambiguous parents, invented categories, anything in Unplaced, assumed spelling corrections, re-parentings with old and new integer, every newly issued integer. Diff input against output before reporting — count labels in, count labels out.

If a non-English output language is requested, translate all five generated strings: "Other… see CV", "Professional areas", "Unplaced", and the keys next_subindex and competencies. Say so when translating the last two, since a parser may expect fixed key names.