---
description: "Break down current uncommitted changes (staged and unstaged) into a series of clean, atomic commits."
argument-hint: "[TARGET_BRANCH=<name>]"
---

Analyze all current uncommitted changes (staged and unstaged) and break them into a series of logical, atomic commits on the current branch or a new target branch. Suitable for maintaining a clean, readable project history.

Steps
1) Analyze workspace
- Scan all modified, new, and deleted files
- Group related changes into logical, self-contained units
- Identify dependencies between changes to determine commit order

2) Create target branch (Optional)
- If a target branch name is provided, create and switch to it before committing
- Otherwise, apply commits to the current branch

3) Plan atomic sequence
- Each commit should represent one coherent idea or functional change
- Order commits so the codebase remains stable/compilable at each step
- Ensure the sequence tells a clear "story" of the implementation

4) Execute commits
- Stage specific lines or files for the first logical unit
- Commit with a clear, descriptive message (Header + Body)
- Use `git commit --no-verify` for intermediate steps if necessary
- Repeat until all changes are committed

5) Verify integrity
- Ensure no changes remain in the working directory
- Confirm the final state matches the original modified state
- Run a final validation check (e.g., linting or tests) on the last commit

Rules
- No AI attribution
- Do not add self as author/contributor
- Each commit must be atomic (one single responsibility)
- Final state must be identical to the state of the files before the process started
