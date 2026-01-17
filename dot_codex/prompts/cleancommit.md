---
description: "Reimplement current branch on fresh branch off main with clean, narrative commit history."
argument-hint: "[SOURCE_BRANCH=<name>] [NEW_BRANCH=<name>]"
---

Reimplement current branch on fresh branch off `main` with clean, narrative-quality commit history. Suitable for reviewer comprehension.

Steps
1) Validate source branch
- No uncommitted changes, no merge conflicts
- Up to date with `main`

2) Analyze diff
- Study changes between source branch and `main`
- Understand final intended state

3) Create clean branch
- New branch off `main`
- Name: `{source_branch}-clean` unless user provides name

4) Plan commit storyline
- Self-contained logical steps
- Each step reads like tutorial stage

5) Reimplement work
- Recreate changes, commit step by step
- Each commit: one coherent idea, clear message + description
- Follow `git-commit` skill message guidance
- Use `git commit --no-verify` for intermediate commits

6) Verify correctness
- Final state exactly matches source branch
- Final commit without `--no-verify`

Rules
- No AI attribution
- Do not add self as author/contributor
- End state identical to source branch
