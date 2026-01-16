# Claude Code Custom Commands - Setup Guide

Custom slash commands for Claude Code that integrate Jira and GitHub into your development workflow.

## Available Commands

| Command | Description |
|---------|-------------|
| `/jira-analyze <TICKET-ID>` | Analyze a Jira ticket, investigate the codebase, propose a fix, and create a PR |
| `/jira-branch <TICKET-ID>` | Create a standardized git branch from a Jira ticket |
| `/jira-devtask` | Generate a dev-task or re-test comment based on current branch changes |

## Prerequisites

### 1. Git

Git is required for all branch and commit operations.

**Installation:**

- **Windows:** Download from [git-scm.com](https://git-scm.com/download/win) or use `winget install Git.Git`
- **macOS:** `brew install git`
- **Linux:** `sudo apt install git` (Debian/Ubuntu) or `sudo dnf install git` (Fedora)

**Verify installation:**
```bash
git --version
```

### 2. GitHub CLI (gh)

Required for creating pull requests and pushing branches to remote.

**Installation:**

- **Windows:** `winget install GitHub.cli`
- **macOS:** `brew install gh`
- **Linux:** See [GitHub CLI installation docs](https://github.com/cli/cli/blob/trunk/docs/install_linux.md)

**Verify installation:**
```bash
gh --version
```

**Authentication:**
```bash
gh auth login
```
Follow the prompts to authenticate with your GitHub account.

### 3. Jira CLI

Required for fetching ticket details, creating dev-tasks, and adding comments.

**Repository:** [https://github.com/ankitpokhrel/jira-cli](https://github.com/ankitpokhrel/jira-cli)

**Installation:**

- **Windows:** `winget install ankitpokhrel.jira-cli` or download from [releases](https://github.com/ankitpokhrel/jira-cli/releases)
- **macOS:** `brew install ankitpokhrel/jira-cli/jira-cli`
- **Linux:**
  ```bash
  # Using Homebrew
  brew install ankitpokhrel/jira-cli/jira-cli

  # Or download binary from releases
  ```

**Verify installation:**
```bash
jira version
```

**Configuration:**

Run the init command to configure your Jira instance:
```bash
jira init
```

You will be prompted for:
- **Jira Server URL:** Your Jira instance (e.g., `https://jira.cignium.com`)
- **Authentication:** Choose between API token or Personal Access Token (PAT)
- **Default Project:** Your default project key (e.g., `DATA`)

**Generate API Token:**
1. Go to your Jira instance → Profile → Personal Access Tokens
2. Create a new token with appropriate permissions
3. Use this token during `jira init`

## Installation

1. Clone this repository to your Claude Code commands directory:
   ```bash
   git clone <repo-url> ~/.claude/commands
   ```

   Or if you already have a commands directory, copy the `.md` files:
   ```bash
   cp jira-analyze.md jira-branch.md jira-devtask.md ~/.claude/commands/
   ```

2. Verify the commands are available in Claude Code by typing `/jira-` and checking autocomplete.

## Usage Examples

### Analyze a Bug Ticket
```
/jira-analyze DATA-15127
```
Claude will fetch the ticket, analyze your codebase, identify the bug, propose a fix, and create a PR after your approval.

### Create a Branch from Ticket
```
/jira-branch DATA-15127
```
Claude will generate a branch name like `data-15127-fix-invalid-age-range` and offer to create it locally and/or push to remote.

### Generate Dev Task or Re-test Comment
```
/jira-devtask
```
Claude will analyze your current branch's commits and either:
- Create a dev-task (for Stories/Tasks)
- Add a re-test comment (for Defects)

## Troubleshooting

### Jira CLI: "Not authenticated"
Run `jira init` again or check your token hasn't expired.

### GitHub CLI: "Not logged in"
Run `gh auth login` to re-authenticate.

### Command not found
Ensure the CLI tools are in your system PATH. Restart your terminal after installation.

### Windows: `findstr` not found
The commands use Windows-specific `findstr`. If using WSL or Git Bash, you may need to modify `jira-branch.md` to use `grep` instead.

## Notes

- These commands include confirmation steps before making changes to your codebase or Jira tickets
- The default main branch for `jira-devtask` is set to `imported_cold_transfer` - modify as needed for your project
- The default entry point for code analysis is `qual_agent.py` - modify in `jira-analyze.md` for different projects
