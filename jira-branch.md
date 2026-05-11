---
model: claude-haiku-4-5-20251001
---

For ticket $ARGUMENTS:

1. **Determine base branch** — Check project memory (MEMORY.md and memory files) for a saved base branch preference:
   - If found: use it silently (no prompt).
   - If not found:
     - Run `git remote show origin | findstr "HEAD branch"` to detect the default branch.
     - Ask the user to confirm, offering the detected branch as the recommended option plus a "Specify different branch" option.
     - Save the confirmed branch to project memory for future use.

2. Run `jira issue view $ARGUMENTS --plain` to get the ticket title.
   - Extract the ticket code from the URL or argument (e.g., DATA-18532).

3. From the title, generate a short summary (max 5-6 relevant keywords).

4. Create the branch name with this exact format:
   - All lowercase
   - Spaces replaced by hyphens
   - No special characters (remove brackets, parentheses, etc.)
   - Format: {ticket-code}-{summary}
   - Example: data-14961-invalid-age-range-and-closing-message

5. Show the proposed branch name and ask:
   - **Proceed — Create locally and remotely (Recommended)**
   - **Reject**: Reject the branch name and ask for a new one

6. If approved, run in sequence:
   ```bash
   git fetch origin
   git stash --include-untracked
   git checkout -b <branch-name> origin/<base-branch>
   git stash pop
   git push -u origin <branch-name>
   ```
   If `git stash` reports no changes to stash, skip stash/pop commands.

   If rejected, ask the user for a new branch name and go back to step 5.
