---
name: ai-prose-review
description: Review markdown prose in the current location against the 21 writing rules from yzhao062/agent-style. Use when the user asks to review, audit, or check docs/READMEs/blog posts for AI tells, robotic phrasing, telltale AI patterns, or wants a prose quality pass. Triggers on /ai-prose-review.
license: MIT
user-invocable: true
allowed-tools:
  - Read
  - Glob
  - Grep
  - Edit
  - Bash
args:
  - name: target
    description: Optional path or glob to scope the review. Defaults to all *.md and *.mdx files in the current working directory.
    required: false
---

> Based on the excellent writing-rules work by Yue Zhao (USC).
> Original: <https://github.com/yzhao062/agent-style>
> This is my own simpler personal version of those rules, trimmed to a single-file review skill I can run on demand.

# AI Prose Review

Audit markdown files in the current location against a checklist of 21 rules drawn from style authorities (Strunk & White, Orwell, Pinker, Gopen & Swan) and field-observed AI-tell patterns. Report violations first; only edit files after the user confirms.

## When to use

- The user invokes `/ai-prose-review` (with or without a path argument).
- The user asks to "review my docs", "audit this README", "check for AI tells", or otherwise wants a prose-quality pass.
- The user pastes prose and asks for a style review against the agent-style rules.

Skip this skill for fiction, poetry, marketing copy, or narrative writing where rhythm and affect matter more than precision. The rules below target technical prose: API docs, design docs, READMEs, runbooks, commit messages, blog posts.

## Process

### 1. Resolve targets

- If the user passed a `target` argument, use it directly. A path means that one file. A glob means expand it.
- Otherwise glob `**/*.md` and `**/*.mdx` from the current working directory.
- Exclude `node_modules/`, `.git/`, `vendor/`, `dist/`, `build/`, `.next/`, `.venv/`.
- If zero files match, stop and tell the user. Do not invent files.

Use `Glob` for discovery. Use `Read` for each file.

### 2. Walk each file against the rules

For every file, read the full contents and check each of the 21 rules in the checklist below. Skip:

- Fenced code blocks (```` ``` ````).
- YAML frontmatter (between leading `---` lines).
- Inline code spans (backticks).
- Raw HTML blocks.

For every violation, record: file path, line number, rule ID, the offending excerpt (short), and a one-sentence "why" so the user can decide whether to fix.

### 3. Report findings

Group by file, ordered by severity. Use a compact table per file:

```
## docs/architecture.md (7 violations)

| Line | Rule    | Severity | Excerpt                                | Why                                  |
|------|---------|----------|----------------------------------------|--------------------------------------|
| 14   | RULE-H  | High     | "studies show that..."                 | Unsupported claim, no citation       |
| 22   | RULE-12 | Medium   | 38-word sentence starting "The system" | Over the 30-word ceiling; split it   |
| 31   | RULE-D  | Low      | "Furthermore," (3rd para in a row)     | Transition-word opener overused      |
```

After the per-file tables, give a one-line summary: total files, total violations, breakdown by severity.

### 4. Ask before editing

Stop after the report. Ask the user which violations to fix:

- All violations across all files.
- A specific file or files.
- A specific rule or set of rules (e.g. "just the high-severity ones").
- None — report only.

Do not pre-emptively rewrite anything.

### 5. Apply fixes

If the user confirms, work one file at a time. Use `Edit` for each violation. Show the diff. For sentence-level rewrites (RULE-12 splits, RULE-02 voice changes), keep the rewrite minimal — preserve the author's voice and meaning. Do not introduce new content or change the technical claims.

After fixes, offer to re-run the review on the changed files to confirm violations are resolved.

## The 21 rules

Each rule lists a short name, a "spot it by" cue, a "fix by" cue, and severity.

### Canonical (RULE-01 to RULE-12)

Source: Strunk & White, Orwell, Pinker, Gopen & Swan.

- **RULE-01 — Curse of knowledge.** *Spot:* unexplained acronyms, undefined terms, leaps that assume reader background. *Fix:* define on first use, add one-line context. **High** when the gap blocks comprehension; **Medium** otherwise.
- **RULE-02 — Passive voice when agency matters.** *Spot:* "was performed by", "is being processed", "has been deployed". *Fix:* name the actor, switch to active. **Medium**.
- **RULE-03 — Concrete over abstract.** *Spot:* "leverages capabilities", "utilises functionality", "various approaches". *Fix:* name the specific thing. **Medium**.
- **RULE-04 — Omit needless words.** *Spot:* "in order to", "due to the fact that", "at this point in time", "it is important to note that". *Fix:* delete or replace with the shorter form. **Medium**.
- **RULE-05 — Dying metaphors and prefab phrases.** *Spot:* "cutting-edge", "best-of-breed", "move the needle", "low-hanging fruit", "deep dive". *Fix:* say what you actually mean. **Low**.
- **RULE-06 — Avoidable jargon.** *Spot:* internal initialisms or domain jargon used without need. *Fix:* swap for everyday English where the precision is not load-bearing. **Medium**.
- **RULE-07 — Affirmative form for affirmative claims.** *Spot:* "not infrequent", "did not fail to", "is not unimportant". *Fix:* state it positively. **Low**.
- **RULE-08 — Match claim strength to evidence.** *Spot:* "always", "never", "guaranteed", "completely eliminates" without proof; or hedging like "may possibly perhaps" when the evidence is solid. *Fix:* tune the verb to the evidence. **High**.
- **RULE-09 — Parallel structure.** *Spot:* lists or coordinated phrases with mismatched grammatical forms ("reading, to write, and edits"). *Fix:* make all items the same shape. **Low**.
- **RULE-10 — Keep related words together.** *Spot:* subject and verb separated by long modifiers; misplaced "only"; dangling clauses. *Fix:* move modifiers next to what they modify. **Medium**.
- **RULE-11 — Stress position.** *Spot:* the most important / newest information buried mid-sentence; sentence ends with a throwaway phrase. *Fix:* move the new or emphasis-bearing information to the end. **Low**.
- **RULE-12 — Sentences over 30 words.** *Spot:* count words; flag any sentence past ~30. *Fix:* split at the natural clause break; vary sentence length across the paragraph. **Medium**.

### Field-observed (RULE-A to RULE-I)

Source: AI-tell patterns logged across LLM output 2022–2026.

- **RULE-A — Over-bulleting.** *Spot:* prose converted into bullet lists where the items are not a genuine list (continuous reasoning, narrative steps). *Fix:* return to paragraph form. **Medium**.
- **RULE-B — Em-dash and en-dash as casual filler.** *Spot:* sentences leaning on `—` or `–` as a generic connector multiple times per paragraph. *Fix:* use a comma, semicolon, or full stop. Reserve dashes for genuine parenthetical breaks. **Low**.
- **RULE-C — Repeated sentence openers.** *Spot:* two or more consecutive sentences starting with the same word or phrase ("The system…", "We…"). *Fix:* vary the opener or restructure. **Low**.
- **RULE-D — Transition-word overuse.** *Spot:* "Additionally,", "Furthermore,", "Moreover,", "In addition,", "It is worth noting that," used as paragraph or sentence openers, especially in succession. *Fix:* drop the transition or replace with a substantive bridge. **Medium**.
- **RULE-E — Summary closer at every paragraph.** *Spot:* every paragraph ending with a "this means…" or "in summary…" line that restates the paragraph. *Fix:* let the paragraph end on its strongest sentence; cut the summary. **Low**.
- **RULE-F — Inconsistent terminology.** *Spot:* the same concept named two ways in one document; an acronym redefined; "user" vs "customer" vs "client" used interchangeably without intent. *Fix:* pick one term and use it everywhere. **High**.
- **RULE-G — Heading case discipline.** *Spot:* mixed sentence-case and title-case section headings within one document. *Fix:* pick one convention. For title case, keep articles and short prepositions lowercase. **Low**.
- **RULE-H — Unsupported factual claims.** *Spot:* "studies show", "research has demonstrated", "it has been proven", a specific number or percentage, a benchmark result, a vendor comparison — all without a citation, link, or in-document evidence. *Fix:* add a citation, link to source data, or weaken the claim to match what you can support. **High** (this is the most important rule).
- **RULE-I — Contractions in formal technical prose.** *Spot:* "don't", "can't", "won't", "it's" in API docs, design docs, research papers, grant proposals. *Fix:* expand to the full form. **Low** (and skip in blog posts or runbooks where casual register is fine — confirm with the user if uncertain).

## Severity tiers (summary)

- **High:** RULE-H, RULE-08, RULE-F, sometimes RULE-01.
- **Medium:** RULE-02, RULE-03, RULE-04, RULE-06, RULE-10, RULE-12, RULE-A, RULE-D.
- **Low:** RULE-05, RULE-07, RULE-09, RULE-11, RULE-B, RULE-C, RULE-E, RULE-G, RULE-I.

If the user asks for a quick pass, run only High and Medium.

## What not to flag

- Code blocks, inline code, frontmatter, raw HTML.
- Direct quotations from another source.
- Generated content the user marks with a comment like `<!-- ai-prose-review: skip -->` to the next blank line.
- Files outside the resolved target set.

If the document is fiction, marketing copy, or narrative writing, stop and confirm with the user before reviewing. The rules target technical prose and will produce noisy results elsewhere.

## Output template

When reporting, follow this shape:

```
# AI Prose Review

Reviewed N files. Found X violations: H high · M medium · L low.

## path/to/file-1.md (V violations)

| Line | Rule    | Severity | Excerpt | Why |
|------|---------|----------|---------|-----|
| ...  | ...     | ...      | ...     | ... |

## path/to/file-2.md (V violations)

...

---

Which would you like to fix?
- All violations
- A specific file
- A specific rule (or just High severity)
- None — report only
```

Keep excerpts short (under ~80 chars). Truncate with `…` if needed. Always include the line number so the user can jump to the source.
