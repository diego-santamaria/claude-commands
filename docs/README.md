# Claude Code Custom Commands - Setup Guide

Custom slash commands for Claude Code that integrate Jira, GitHub, and Kubernetes into your development workflow.

## Available Commands

| Command | Description |
|---------|-------------|
| `/jira-analyze <TICKET-ID>` | Analyze a Jira ticket, investigate the codebase, propose a fix, and create a PR |
| `/jira-branch <TICKET-ID>` | Create a standardized git branch from a Jira ticket |
| `/jira-devtask` | Generate a dev-task or re-test comment based on current branch changes |
| `/jira-from-example <EXAMPLE-TICKET-ID>` | Create a new Jira ticket by copying metadata from an existing ticket |
| `/k8s-logs <pod-name-or-keyword>` | Troubleshoot Kubernetes pod logs with guided error filtering and scaling analysis |

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

Required for fetching ticket details, creating dev-tasks, adding comments, and creating tickets from examples.

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
- **Jira Server URL:** Your Jira instance (e.g., `https://your-company.atlassian.net`)
- **Authentication:** Choose between API token or Personal Access Token (PAT)
- **Default Project:** Your default project key (e.g., `PROJ`)

**Generate API Token:**
1. Go to your Jira instance → Profile → Personal Access Tokens (or API Tokens for Atlassian Cloud)
2. Create a new token with appropriate permissions
3. Use this token during `jira init`

### 4. kubectl (Kubernetes CLI)

Required for the `/k8s-logs` command. Used to query pod logs, events, deployments, and scaling.

**Installation:**

- **Windows:** `winget install Kubernetes.kubectl` or via Docker Desktop (includes kubectl)
- **macOS:** `brew install kubectl`
- **Linux:**
  ```bash
  curl -LO "https://dl.k8s.io/release/$(curl -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
  chmod +x kubectl && sudo mv kubectl /usr/local/bin/
  ```
  Or via package manager: `sudo apt install kubectl` (Debian/Ubuntu)

**Verify installation:**
```bash
kubectl version --client
```

**Configuration:**

kubectl requires a valid kubeconfig file pointing to your cluster(s). Your cluster admin or cloud provider will supply this. Common locations:
- Default: `~/.kube/config`
- Override: `export KUBECONFIG=/path/to/your/config`

Verify you can reach your cluster:
```bash
kubectl config current-context
kubectl get nodes
```

> **Important:** The `/k8s-logs` command always asks you to confirm the active cluster context before running any operations. Never skip this confirmation — it prevents accidental commands against production.

## Installation

1. Clone this repository to your Claude Code commands directory:
   ```bash
   git clone https://github.com/diego-santamaria/claude-commands.git ~/.claude/commands
   ```

   Or if you already have a commands directory, copy the `.md` files:
   ```bash
   cp jira-analyze.md jira-branch.md jira-devtask.md jira-from-example.md k8s-logs.md ~/.claude/commands/
   ```

2. Verify the commands are available in Claude Code by typing `/jira-` or `/k8s-` and checking autocomplete.

## Configuration

Some commands reference configurable values. Update these in the command files as needed:

| Value | File | Description |
|-------|------|-------------|
| `JIRA_BASE_URL` | jira-devtask.md | Your Jira instance URL |
| `MAIN_BRANCH` | jira-devtask.md | Your repository's main branch |

## Usage Examples

### Analyze a Bug Ticket
```
/jira-analyze PROJ-123
```
Claude will fetch the ticket, analyze your codebase, identify the bug, propose a fix, and create a PR after your approval.

### Create a Branch from Ticket
```
/jira-branch PROJ-123
```
Claude will generate a branch name like `proj-123-fix-login-validation` and offer to create it locally and/or push to remote.

### Generate Dev Task or Re-test Comment
```
/jira-devtask
```
Claude will analyze your current branch's commits and either:
- Create a dev-task (for Stories/Tasks)
- Add a re-test comment (for Defects)

### Create a Ticket from an Example
```
/jira-from-example PROJ-100
```
Claude fetches metadata (project, type, priority, assignee, epic, sprint, labels) from `PROJ-100` and applies it to a new ticket you describe. You provide the title and problem; Claude formats the description according to the issue type (Bug or Story).

### Troubleshoot Kubernetes Pod Logs
```
/k8s-logs my-service
```
Claude will:
1. Show your current cluster context and ask for confirmation before proceeding
2. Find matching pods across all namespaces
3. Pull and filter logs for errors, warnings, and capacity issues
4. Check replica counts and horizontal pod autoscaler (HPA) status
5. Surface Kubernetes events for the namespace

## Troubleshooting

### Jira CLI: "Not authenticated"
Run `jira init` again or check your token hasn't expired.

### GitHub CLI: "Not logged in"
Run `gh auth login` to re-authenticate.

### kubectl: "Unable to connect to the server"
Check that your kubeconfig is correctly set up: `kubectl config view`. Contact your cluster admin if the context is missing.

### kubectl: "error: the server doesn't have a resource type"
You may be connected to the wrong cluster context. Run `kubectl config get-contexts` to list available contexts and `kubectl config use-context <name>` to switch.

### Command not found
Ensure the CLI tools are in your system PATH. Restart your terminal after installation.

### Windows: `findstr` vs `grep`
The `jira-branch.md` command uses Windows `findstr`. If using WSL or Git Bash, replace `findstr` with `grep`.

## Customization

These commands are templates designed to be adapted to your workflow:

- **Entry points:** `jira-analyze.md` will ask for guidance if it can't detect the entry point automatically
- **Branch naming:** Modify `jira-branch.md` to change the branch naming convention
- **Ticket types:** `jira-devtask.md` handles Defects differently from Stories/Tasks - adjust as needed
- **Jira fields:** `jira-from-example.md` copies common custom fields (epic, sprint, team) — extend the field list as needed for your Jira configuration

## Contributing

Feel free to fork and adapt these commands for your own workflow. Pull requests welcome!

## License

MIT License - Use freely and adapt to your needs.
