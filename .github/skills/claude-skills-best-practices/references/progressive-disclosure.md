# Progressive Disclosure Patterns

## Contents
- Purpose and model
- Pattern 1: High-level guide with references
- Pattern 2: Domain-specific organization
- Pattern 3: Conditional details
- Rules and anti-patterns
- Reference file structure checklist

## Purpose and model

Progressive disclosure keeps core instructions in SKILL.md and defers details into reference files that are only loaded when needed. This preserves context budget and reduces distraction for unrelated tasks.

## Pattern 1: High-level guide with references

Use SKILL.md as a table of contents and link to deeper files:

- Advanced features → references/advanced.md
- API reference → references/reference.md
- Examples → references/examples.md

Claude only reads the linked files when the user’s request requires them.

## Pattern 2: Domain-specific organization

Split knowledge by domain so Claude loads only the relevant slice:

reference/
  finance.md
  sales.md
  product.md

Example navigation from SKILL.md:
- Finance metrics → references/finance.md
- Sales metrics → references/sales.md
- Product metrics → references/product.md

## Pattern 3: Conditional details

Link to specialized content only when a feature is requested:

- Tracked changes → references/redlining.md
- OOXML internals → references/ooxml.md

## Rules and anti-patterns

- One level deep only: SKILL.md should link directly to reference files.
- Avoid reference → reference chains. They cause partial reads and lost context.
- Add a table of contents to any reference file longer than ~100 lines.
- Use forward slashes in paths on all platforms.
- Make intent explicit: run vs read.

## Reference file structure checklist

- Clear title
- Table of contents (if long)
- Sectioned content with headings
- Concrete examples or commands
- Explicit links back to SKILL.md if needed
