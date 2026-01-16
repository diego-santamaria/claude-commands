For ticket $ARGUMENTS:

## Phase 1: Gather Information

1. Verify the ticket exists and get full details:
   ```bash
   jira issue view $ARGUMENTS
   ```
   If the command fails, inform me that the ticket was not found and stop.

2. Extract and analyze all available information:
   - Summary/Title
   - Description
   - Steps to reproduce
   - Expected behavior
   - Actual behavior
   - Any comments from ITQE or other team members
   - Acceptance criteria (if any)
   - Screenshots (note what they show if mentioned)
   - Error logs (capture exact error messages, stack traces)
   - Links to the web scenario where the error occurs
   - Environment details (staging, prod, etc.)

## Phase 2: Confirm Understanding

3. Present a summary of what you understood about the bug/defect:
   - What is the problem?
   - When does it occur? (conditions/triggers)
   - What is the expected behavior?
   - What is the actual/incorrect behavior?
   - Relevant error messages or logs
   - URL/scenario where it happens (if provided)

4. Ask me to confirm if your understanding is correct before proceeding. Wait for my confirmation.

## Phase 3: Code Analysis

5. Once confirmed, start analyzing the codebase:
   - Detect the entry point automatically by:
     - Looking at error logs/stack traces for file references
     - Searching for keywords from the bug description in the codebase
     - Identifying the relevant module based on the feature/flow affected
   - If there isn't enough information to determine the entry point automatically, default to `qual_agent.py`
   - Trace the relevant code paths based on the bug description
   - Identify the root cause of the issue
   - Look for related files that may be affected

6. Present your findings:
   - Which file(s) contain the bug?
   - How did you identify the entry point?
   - What is the root cause?
   - Show the problematic code snippet(s)

## Phase 4: Propose Solution

7. Propose a solution:
   - Explain the fix in plain terms
   - Show the code changes needed (before/after)
   - Mention any potential side effects or risks
   - Suggest any additional tests if applicable

8. Ask for my approval before making changes. Wait for my confirmation.

## Phase 5: Implement and Create PR

9. Once approved, implement the fix:
   ```bash
   # Make the code changes to the identified files
   ```

10. Stage and commit the changes:
    ```bash
    git add .
    git commit -m "<ticket-code>: <short description of fix>"
    ```
    - Commit message format: `TICKET-123: Fix brief description`
    - Keep it concise but descriptive

11. Push the branch:
    ```bash
    git push origin <current-branch>
    ```

12. Create a Pull Request using GitHub CLI:
    ```bash
    gh pr create --title "<ticket-code>: <title>" --body "<description>"
    ```
    - Title: `TICKET-123: Brief description of the fix`
    - Body should include:
      - **Problem**: What was the bug?
      - **Root Cause**: What caused it?
      - **Solution**: What was changed and why?
      - **Testing**: How to verify the fix?
      - **Jira Ticket**: Link to the ticket

13. Present the PR URL to me.

## Important Notes

- Always wait for confirmation at steps 4, 8 before proceeding
- If at any point you're unsure about the fix, say so and ask for guidance
- If the bug seems too complex or risky, recommend breaking it down or pairing with another dev
- Do not modify any code without explicit approval
