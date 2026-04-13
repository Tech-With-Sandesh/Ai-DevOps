# Day 2 — How It Works
> Understanding the concepts behind the GitHub AI Agent

---

## What Changed from Day 1

```
Day 1                          Day 2
──────────────────────         ──────────────────────────
You wrote every tool     →     MCP gives tools instantly
AWS operations           →     GitHub operations
LangChain AgentExecutor  →     LangGraph (new standard)
Fixed tasks in code      →     Plain English input
6 tools                  →     26 tools
```

---

## What is MCP

MCP stands for **Model Context Protocol**.

It is an open standard that lets anyone build pre-made tool servers
that any AI agent can plug into instantly.

```
Without MCP:
You write @tool functions manually
You handle API authentication
You parse every API response
You maintain the code forever

With MCP:
Someone builds the tool server once
You plug in with 3 lines of config
Instant access to all tools
Community maintains it
```

Think of MCP like a USB standard.
Any USB device works with any USB port.
Any MCP server works with any AI agent.

---

## GitHub MCP Server

The GitHub MCP server is an official pre-built tool server
that gives your agent 26 GitHub tools instantly.

```
Your Agent
    │
    ▼
MultiServerMCPClient  ←  connects to
    │
    ▼
GitHub MCP Server     ←  runs as a local process (npx)
    │
    ▼
GitHub API            ←  real GitHub operations
```

You never call the GitHub API directly.
The MCP server handles authentication, requests, and responses.

---

## The 26 GitHub MCP Tools

```
REPOSITORIES
create_repository       →  create a new repo
search_repositories     →  find repos on GitHub
fork_repository         →  fork any repo

FILES
create_or_update_file   →  create or edit a file
get_file_contents       →  read any file
push_files              →  push multiple files at once

BRANCHES
create_branch           →  create a new branch
list_commits            →  see commit history

PULL REQUESTS
create_pull_request     →  open a new PR
list_pull_requests      →  list open PRs
get_pull_request        →  get PR details
merge_pull_request      →  merge a PR
create_pull_request_review → review a PR
get_pull_request_files  →  see changed files
get_pull_request_status →  check CI status
update_pull_request_branch → sync with base
get_pull_request_comments  → read comments
get_pull_request_reviews   → read reviews

ISSUES
create_issue            →  open a new issue
list_issues             →  list all issues
get_issue               →  get issue details
update_issue            →  edit or close issue
add_issue_comment       →  comment on issue

SEARCH
search_code             →  search code in repos
search_issues           →  search issues and PRs
search_users            →  find GitHub users
```

---

## LangGraph vs LangChain AgentExecutor

In Day 1 we used LangChain's `AgentExecutor`.
In Day 2 we use LangGraph's `create_react_agent`.

```
LangChain AgentExecutor (old)
──────────────────────────────
Simple linear loop
Limited control over flow
Being deprecated in newer versions

LangGraph create_react_agent (new)
──────────────────────────────────
Graph-based execution
More reliable tool calling
Handles complex multi-step tasks
Active development — future standard
```

Same ReAct loop underneath. Better engine on top.

---

## How the Agent Reasons

When you type:
```
create a pull request from feature/day2 into main
```

The agent does this internally:

```
Thought:  I need to create a pull request.
          I have create_pull_request tool available.
          I need: repo owner, repo name, head branch, base branch, title.
          Owner and repo are in my context.
          Head = feature/day2, base = main.

Action:   create_pull_request
Input:    {owner, repo, head, base, title}

Observation: PR #3 created successfully

Thought:  Task complete. I have the PR URL and number.

Answer:   PR #3 created: github.com/user/repo/pull/3
```

You typed one sentence.
The agent figured out every parameter on its own.

---

## Multiline Input — How It Works

Standard `input()` in Python reads one line and stops.

We fixed this with a loop:

```python
lines = []
while True:
    line = input()
    if line.strip().upper() == "DONE":  # stop signal
        break
    lines.append(line)

task = "\n".join(lines)  # join all lines into one task
```

Type as many lines as you want.
Type DONE on a new line to submit.
Agent receives the full multiline task.

---

## Why Plain English Works

The agent works in plain English because:

```
1. Gemini was trained on billions of GitHub-related texts
   — it understands GitHub concepts natively

2. Each MCP tool has a clear description
   — agent reads descriptions to pick the right tool

3. ReAct loop lets agent ask for missing info
   — it figures out parameters from context

4. Your username is always in the system prompt
   — agent knows who it's acting for
```

---

## The Full GitHub Lifecycle

```
Create Repo
    │
    ▼
Create Branch
    │
    ▼
Push Files to Branch
    │
    ▼
Open Pull Request
    │
    ▼
Review + Comment on PR
    │
    ▼
Merge PR
    │
    ▼
Delete Branch
    │
    ▼
Create Issue
    │
    ▼
Comment on Issue
    │
    ▼
Close Issue
```

One agent. Plain English. Every step automated.

---

## Your Role as DevOps Engineer

```
What the agent does:
✅ Picks the right tool
✅ Figures out API parameters
✅ Handles authentication
✅ Executes the operation
✅ Reports back clearly

What you do:
✅ Define the goal clearly
✅ Verify the output on GitHub
✅ Catch mistakes agent makes
✅ Design the workflow
✅ Add guardrails for production use
```

The agent is fast. You are accurate.
Together — unstoppable.

---

## What's Different About This Approach

```
Traditional way:
→ Open GitHub UI and click through everything
→ Or learn GitHub CLI commands
→ Or write scripts with PyGitHub library
→ Handle auth, pagination, error handling yourself

AI Agent way:
→ Type what you want in plain English
→ Agent picks the right tool
→ Agent handles everything underneath
→ You verify the result
```

Same outcome. Fraction of the effort.

---

## Key Concepts Summary

| Concept | What it is |
|---|---|
| MCP | Standard for pre-built agent tool servers |
| GitHub MCP Server | 26 GitHub tools via npx |
| MultiServerMCPClient | Python client that connects to MCP servers |
| LangGraph | Modern agent execution framework |
| create_react_agent | LangGraph's ReAct agent — replacement for AgentExecutor |
| ReAct loop | Think → Act → Observe → Repeat |
| stdio transport | How agent communicates with MCP server locally |

---

## What's Next — Day 3

Day 2 was one agent doing GitHub operations.

Day 3 is two agents working together:

```
Orchestrator Agent
      │
      ├── GitHub Agent  ←  monitors Actions workflows
      │
      └── AWS Agent     ←  manages infrastructure

Real scenario:
GitHub Actions workflow fails
→ GitHub agent detects it
→ Tells AWS agent
→ AWS agent rolls back the deployment
→ Zero human intervention
```

That's multi-agent systems. Coming in Day 3.
