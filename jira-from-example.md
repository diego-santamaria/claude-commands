When the user asks you to create a Jira ticket and provides an example ticket ($ARGUMENTS) to copy metadata from:

## Step 1 — Fetch the example ticket

Run both in parallel:
```bash
jira issue view <EXAMPLE_TICKET> --plain
jira issue view <EXAMPLE_TICKET> --raw
```

Extract these fields to copy verbatim onto the new ticket:
- **Project** (`project.key`)
- **Issue type** (Bug / Story / Task)
- **Priority**
- **Assignee** (display name)
- **Reporter** (display name)
- **Epic link** (`customfield_10014`) — only if set
- **Sprint** (`customfield_10020`) — only if set
- **Team** (`customfield_10001`) — only if set
- **Labels**
- **Components**

## Step 2 — Frame the description based on issue type

**If Bug:**
```
*Steps to reproduce:*
• <step 1>
• <step 2>
• ...

*Current result:*
• <what actually happens>

*Expected result:*
• <what should happen>
```

**If Story (new feature):**
```
As a Software Engineer, I want to <goal> so that <benefit>.

*Acceptance Criteria:*
• <criterion 1>
• <criterion 2>
```

## Step 3 — Create the ticket

```bash
jira issue create \
  -p <PROJECT> \
  -t <TYPE> \
  -y <PRIORITY> \
  -a "<ASSIGNEE>" \
  -r "<REPORTER>" \
  --custom "customfield_10001=<TEAM>" \
  -s "<TITLE>" \
  -b "<DESCRIPTION>" \
  --no-input
```

Add `--custom "customfield_10014=<EPIC>"` and `--custom "customfield_10020=<SPRINT_ID>"` only if those fields were set on the example ticket.

## Rules
- Always prefix the title with the project tag the user specifies (e.g. `[PROJECT TAG]`)
- Never copy the title or description from the example — only its metadata
- Bug → Steps to Reproduce / Current Result / Expected Result
- Story → "As a Software Engineer, I want..."
- Print the created ticket URL at the end
