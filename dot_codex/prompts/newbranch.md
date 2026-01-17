---
description: Create a new branch off a specific base branch and switch to it
argument-hint: <base_branch> <new_branch_name>
---

Perform the following git operations:
1. Checkout the base branch: `$1`.
2. Ensure the base branch is up to date by pulling the latest changes.
3. Create and checkout a new branch named `$2` based on `$1`.
