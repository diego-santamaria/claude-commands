# Generate Jira Dev Task

Generate a Jira dev-task (child ticket) based on the changes made on the current git branch. If the parent ticket is a Defect, add a re-test comment instead.

## Configuration

Before using this command, set these values for your environment:
- **JIRA_BASE_URL**: Your Jira instance URL (e.g., `https://your-company.atlassian.net`)
- **MAIN_BRANCH**: Your repository's main branch (e.g., `main`, `master`, `develop`)

## Instructions

### Step 1: Extract Parent Ticket

1. Get the current branch name:
   ```
   git branch --show-current
   ```

2. Extract the ticket ID from the beginning of the branch name (e.g., `DATA-15127` from `data-15127-add-invoca-genesys-ids-transfer-metadata`). The pattern is typically `<PROJECT>-<NUMBER>` (case-insensitive).

3. **If ticket ID is found in the branch name:**
   - Present the parent ticket URL for user confirmation:
     ```
     Parent ticket: <JIRA_BASE_URL>/browse/<TICKET-ID>
     ```
   - **Wait for user confirmation before proceeding.**

4. **If ticket ID is NOT found in the branch name:**
   - Ask the user to provide the parent ticket ID
   - Once provided, proceed directly without additional confirmation (since the user just provided it)

### Step 2: Get Ticket Details and Analyze Changes

1. Get the parent ticket details to determine the issue type and reporter:
   ```
   jira issue view <TICKET-ID> --plain
   ```

2. Get the main branch (detect automatically via `git remote show origin | grep 'HEAD branch'` or use MAIN_BRANCH configuration).

3. Get the list of commits on the current branch:
   ```
   git log <main-branch>..HEAD --oneline
   ```

4. Get the detailed diff of changes:
   ```
   git diff <main-branch>..HEAD
   ```

### Step 3: Generate Content Based on Ticket Type

#### If parent ticket is a Defect:

Generate a **re-test comment** to add to the parent ticket. The comment should:

1. **Tag the reporter** using Jira mention syntax: `[~username]`
2. **Announce readiness for re-test** with varied phrasing (so it looks human-written each time)
3. **Explain the bug and fix** in non-technical terms suitable for ITQE/QA audience

**Comment format:**

```
<Greeting> [~reporter.username], <varied phrase saying ticket is ready for re-test>.

**What was happening:** <Non-technical explanation of the bug from the user's perspective>

**What I fixed:** <Non-technical explanation of the solution and what the user should now experience>

<Optional friendly closing>
```

**Example greeting/readiness phrases (vary these):**
- "Hey [~username], this one is ready for re-test."
- "Hi [~username], I've pushed a fix for this and it´s ready for re-test."
- "[~username], this should be good to go now. Ready for another round of testing."
- "Hey [~username], fix is in! Ready for re-test."

Present the comment to the user and **wait for confirmation before posting**.

#### If parent ticket is NOT a Defect (e.g., Story, Task):

Generate a **dev-task** with title and description in the following format:

**Title:** `<Concise description of what was done>` (do NOT include the parent ticket code)

**Description:** (use markdown format)

```
As a <role>,
I want <what the change accomplishes>,
So that <the business value or benefit>.

*Acceptance Criteria:*
 * <Specific verifiable criterion>
 * <Another criterion>

*Technical Changes:*
 * <Specific technical change with file name>
 * <Another technical change>
```

Present the title and description to the user and **wait for confirmation before creating the ticket**.

### Step 4: Create the Ticket or Comment

After user confirms:

#### For Defects (add comment and update status):

1. Add the comment:
   ```
   jira issue comment add <TICKET-ID> $'<COMMENT_BODY>'
   ```
   Note: Use `$'...'` syntax for multi-line comments with proper escaping of single quotes.

2. **Ask the user** if they want to change the ticket status to "Dev Complete". If confirmed:
   ```
   jira issue move <TICKET-ID> "Dev Complete"
   ```

Return confirmation:
```
Comment added to: <JIRA_BASE_URL>/browse/<TICKET-ID>
Status changed to Dev Complete: Yes/No
```

#### For non-Defects (create dev-task):

```
jira issue create --type "Dev Task" --parent <PARENT-TICKET-ID> --summary "<TITLE>" --body "<DESCRIPTION>"
```

Return the full link to the new ticket:
```
New dev-task created: <JIRA_BASE_URL>/browse/<NEW-TICKET-ID>
```

## Guidelines

- The "As a" role should reflect who benefits from the change (typically "software developer" or more specific)
- Acceptance criteria should be specific and testable
- Technical changes should include file names and what was modified
- Keep the title concise and descriptive (under 80 characters)
- Use caret (`^`) notation for special characters in code references: `\{value}`
- For defect comments, keep explanations non-technical - the audience is QA, not developers
- Vary the comment phrasing each time so it doesn't look auto-generated
