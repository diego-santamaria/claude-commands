For ticket $ARGUMENTS:

1. Detect the default branch of the repository:
```bash
   git remote show origin | findstr "HEAD branch"
```

2. Run `jira issue view $ARGUMENTS --plain` to get the ticket title

3. From the title, generate a short summary (max 5-6 relevant keywords)

4. Create the branch name with this exact format:
   - All lowercase
   - Spaces replaced by hyphens
   - No special characters (remove brackets, parentheses, etc.)
   - Format: {ticket-code}-{summary}
   - Example: data-14961-invalid-age-range-and-closing-message

5. Show me the proposed branch name and ask with three options:
   - **Create locally and remotely (Recommended)**: Create the branch locally and push to origin
   - **Create locally only**: Create the branch locally without pushing
   - **Reject**: Reject the branch name and ask for a new one

6. Based on the user's choice:

   **If approved (either local or local+remote)**, run in sequence:
   ```bash
   git fetch origin
   git stash --include-untracked
   git checkout -b <branch-name> origin/<default-branch>
   git stash pop
   ```
   If there are no changes to stash, skip the stash commands.

   **If "Create locally and remotely" was selected**, also run:
   ```bash
   git push -u origin <branch-name>
   ```

   **If rejected**, ask the user for a new branch name and go back to step 5.