# ai-prose-review

A Claude Code skill that reviews markdown prose against a checklist of 21 writing rules — 12 from style authorities (Strunk & White, Orwell, Pinker, Gopen & Swan) and 9 from observed AI-tell patterns.

This is my simpler personal version of [yzhao062/agent-style](https://github.com/yzhao062/agent-style) by Yue Zhao, distilled into a single self-contained `SKILL.md`. The original repo is the canonical source — go there for the research, the full ruleset, and the dual-mode (generation-time and post-draft) implementation. This repo is just the post-draft review skill, kept lean enough that I can run it on demand from any project.

## Install

Symlink the repo into your Claude Code skills directory:

```bash
ln -s "$(pwd)" ~/.claude/skills/ai-prose-review
```

Then in any Claude Code session:

```
/ai-prose-review                       # review all *.md / *.mdx in cwd
/ai-prose-review path/to/file.md       # review one file
/ai-prose-review 'docs/**/*.md'        # review a glob
```

The skill reports violations grouped by file and severity, then asks before editing anything.

## Credit

- Original ruleset and research: [Yue Zhao — agent-style](https://github.com/yzhao062/agent-style) (CC BY 4.0 / MIT)
- This personal-use repackaging: MIT
