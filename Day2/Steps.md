# Episode 2 — GitHub Operations with AI Agent & MCP
> Control GitHub completely using plain English. No API knowledge needed.

---

## What You'll Build
An AI agent that performs complete GitHub operations — creating repos, branches, files, pull requests, issues and more — just by typing what you want in plain English.

---

## How It Works
```
You type:   "create a branch called feature/day2"
Agent:      thinks → picks the right GitHub tool → executes it
Result:     branch created on GitHub instantly
```

The agent uses **GitHub MCP Server** — a pre-built tool server with 26 GitHub tools. You don't write any GitHub API code. Just plug in and use.

---

## Prerequisites
- AWS account with CloudShell access
- Google account (for Gemini API key)
- GitHub account (for GitHub token)

---

## Step 1 — Get Gemini API Key
1. Go to [aistudio.google.com](https://aistudio.google.com)
2. Sign in with Google
3. Click **Get API Key** → **Create API key**
4. Copy the key

---

## Step 2 — Get GitHub Personal Access Token
```
1. github.com → click your avatar → Settings
2. Scroll down → Developer Settings (bottom left)
3. Personal Access Tokens → Tokens (classic)
4. Generate new token (classic)
5. Name: devops-agent
6. Expiration: 90 days
7. Select scopes:
   ✅ repo
   ✅ workflow
   ✅ read:org
8. Click Generate token → Copy it
```

---

## Step 3 — Open AWS CloudShell
```
AWS Console → Click >_ icon (top right) → Wait for it to load
```

---

## Step 4 — Create Project Folder
```bash
mkdir day2 && cd day2
```

---

## Step 5 — Install Python 3.11
MCP requires Python 3.10+. CloudShell has 3.9 by default.
```bash
# Check current version
python3 --version

# If below 3.10 install 3.11
sudo yum install python3.11 -y

# Verify
python3.11 --version
```

---

## Step 6 — Create Virtual Environment
```bash
python3.11 -m venv venv
source venv/bin/activate

# Verify python version inside venv
python3 --version
# Must show 3.11.x
```

---

## Step 7 — Upgrade pip
```bash
python3 -m pip install --upgrade pip
```

---

## Step 8 — Install Packages
```bash
pip install langchain \
            langchain-google-genai \
            langchain-mcp-adapters \
            langchainhub \
            python-dotenv \
            rich
```

---

## Step 9 — Setup npm for GitHub MCP Server
```bash
# Configure npm to install in home directory
mkdir -p ~/.npm-global
npm config set prefix ~/.npm-global
echo 'export PATH=~/.npm-global/bin:$PATH' >> ~/.bashrc
source ~/.bashrc

# Install GitHub MCP server
npm install -g @modelcontextprotocol/server-github

# Verify — you should see this message
npx @modelcontextprotocol/server-github
# Output: GitHub MCP Server running on stdio
# Press Ctrl+C to stop
```

---

## Step 10 — Create .env File
```bash
cat > .env << EOF
GOOGLE_API_KEY=paste_your_gemini_key_here
GITHUB_PERSONAL_ACCESS_TOKEN=paste_your_github_token_here
GITHUB_USERNAME=your_github_username
EOF
```

```bash
# Protect your keys
cat > .gitignore << EOF
.env
venv/
__pycache__/
EOF
```

---

## Step 11 — Verify Everything Works
```bash
# Check keys are loaded
cat > test.py << 'EOF'
from dotenv import load_dotenv
import os
load_dotenv()

gemini = os.getenv("GOOGLE_API_KEY")
github = os.getenv("GITHUB_PERSONAL_ACCESS_TOKEN")
user   = os.getenv("GITHUB_USERNAME")

print(f"✅ Gemini key  : {gemini[:8]}...hidden")
print(f"✅ GitHub token: {github[:8]}...hidden")
print(f"✅ GitHub user : {user}")
EOF

python3 test.py
```

---

## Step 12 — List All GitHub MCP Tools
```bash
cat > list_tools.py << 'EOF'
import asyncio
import os
from dotenv import load_dotenv
from langchain_mcp_adapters.client import MultiServerMCPClient
load_dotenv()

TOKEN = os.getenv("GITHUB_PERSONAL_ACCESS_TOKEN")

async def list_tools():
    client = MultiServerMCPClient(
        {
            "github": {
                "command": "npx",
                "args": ["-y", "@modelcontextprotocol/server-github"],
                "env": {
                    "GITHUB_PERSONAL_ACCESS_TOKEN": TOKEN
                },
                "transport": "stdio"
            }
        }
    )

    tools = await client.get_tools()
    print(f"\n✅ Total GitHub MCP Tools: {len(tools)}")
    print("="*50)
    for i, tool in enumerate(tools, 1):
        print(f"\n{i}. {tool.name}")
        print(f"   {tool.description}")

if __name__ == "__main__":
    asyncio.run(list_tools())
EOF

python3 list_tools.py
```

You should see 26 tools listed.

---

## Step 13 — Create the Agent
```bash
cat > github_agent.py << 'EOF'
import asyncio
import os
from dotenv import load_dotenv
from langchain_google_genai import ChatGoogleGenerativeAI
from langchain_mcp_adapters.client import MultiServerMCPClient
from langgraph.prebuilt import create_react_agent

load_dotenv()

TOKEN    = os.getenv("GITHUB_PERSONAL_ACCESS_TOKEN")
USERNAME = os.getenv("GITHUB_USERNAME")

async def run(task: str):
    client = MultiServerMCPClient(
        {
            "github": {
                "command": "npx",
                "args": ["-y", "@modelcontextprotocol/server-github"],
                "env": {
                    "GITHUB_PERSONAL_ACCESS_TOKEN": TOKEN
                },
                "transport": "stdio"
            }
        }
    )

    tools = await client.get_tools()
    print(f"\n✅ GitHub MCP connected — {len(tools)} tools\n")

    llm = ChatGoogleGenerativeAI(
        model="gemini-3.1-flash-lite-preview",
        temperature=0,
        max_retries=3,
        request_timeout=60
    )

    agent = create_react_agent(llm, tools)

    result = await agent.ainvoke({
        "messages": [
            {
                "role": "user",
                "content": f"""
                You are a senior DevOps engineer.
                GitHub username : {USERNAME}

                Task: {task}
                """
            }
        ]
    })

    final = result["messages"][-1].content

    print("\n" + "="*50)
    print("✅ RESULT")
    print("="*50)
    print(final)

def get_input():
    """Read multiline input — type DONE to submit"""
    print("\n🤖 GitHub AI Agent — Day 2")
    print("="*50)
    print(f"   User : {os.getenv('GITHUB_USERNAME')}")
    print("="*50)
    print("\nType your task below.")
    print("When done type DONE on a new line and press Enter\n")

    lines = []
    while True:
        line = input()
        if line.strip().upper() == "DONE":
            break
        lines.append(line)

    return "\n".join(lines)

if __name__ == "__main__":
    task = get_input()
    print(f"\n👉 Running task...\n")
    asyncio.run(run(task))
EOF
```

---

## Step 14 — Run the Agent
```bash
python3 github_agent.py
```

---

## Step 15 — Full GitHub Flow (One Task at a Time)

Run `python3 github_agent.py` before each task.
Type DONE on a new line to submit.

### Task 1 — Create Repository
```
create a new public repository called ai-devops-series
with description "AI DevOps YouTube Series built with GitHub MCP agent"
and initialize it with a README
DONE
```

### Task 2 — Create Branch
```
create a new branch called feature/ai-devops-day2
from main in ai-devops-series repo
DONE
```

### Task 3 — Create File in Branch
```
create a file called day2.md in branch feature/ai-devops-day2
in ai-devops-series repo with this content:
# Day 2 - GitHub MCP Agent
This file was created by an AI DevOps agent.
Branch: feature/ai-devops-day2
Series: AI DevOps YouTube Series
DONE
```

### Task 4 — Create Pull Request
```
create a pull request in ai-devops-series repo
title: "feat: add day2 notes from AI agent"
from branch feature/ai-devops-day2 into main
DONE
```

### Task 5 — Add PR Comment
```
list all open pull requests in ai-devops-series repo
and add a comment "LGTM - Reviewed and approved by AI agent"
to the first open pull request
DONE
```

### Task 6 — Merge PR
```
merge the first open pull request in ai-devops-series repo
DONE
```

### Task 7 — Delete Branch
```
delete the branch feature/ai-devops-day2 in ai-devops-series repo
DONE
```

### Task 8 — Create Issue
```
create an issue in ai-devops-series repo
title: "Add CloudWatch monitoring guide"
body: "Need documentation for setting up CloudWatch alarms. Priority: High"
DONE
```

### Task 9 — Add Issue Comment
```
add a comment "Will be covered in Day 3 of the series"
to the latest open issue in ai-devops-series repo
DONE
```

### Task 10 — Close Issue
```
close the latest open issue in ai-devops-series repo
with comment "Closing - will track in wiki"
DONE
```

---

## Project Structure
```
day2/
├── .env              ← API keys (never commit)
├── .gitignore        ← protects .env
├── test.py           ← verify keys
├── list_tools.py     ← shows all 26 GitHub tools
├── github_agent.py   ← main agent
└── venv/             ← Python 3.11 environment
```

---

## Available GitHub MCP Tools

```
Repositories  →  create_repository, search_repositories
Files         →  create_or_update_file, get_file_contents, push_files
Branches      →  create_branch, list_commits
Pull Requests →  create_pull_request, list_pull_requests,
                 get_pull_request, merge_pull_request,
                 create_pull_request_review, add_pr_comment
Issues        →  create_issue, list_issues, get_issue,
                 update_issue, add_issue_comment
Search        →  search_code, search_issues, search_users
Forks         →  fork_repository
```

---

## Troubleshooting

| Error | Fix |
|---|---|
| `ModuleNotFoundError` | Run `source venv/bin/activate` |
| `langchain-mcp-adapters not found` | Upgrade pip first: `python3 -m pip install --upgrade pip` |
| `Python version error` | Install python3.11: `sudo yum install python3.11 -y` |
| `npm EACCES permission` | Run `npm config set prefix ~/.npm-global` then add to PATH |
| `Rate limit hit` | Switch model to `gemini-3.1-flash-lite-preview` — 500 RPD free |
| `Reference already exists` | Branch exists already — just create file directly on it |
| `No space left on device` | Run `rm -rf ~/.cache/pip && pip cache purge` |

---

## Gemini Free Tier — Best Models for This Series

| Model | RPD | Use for |
|---|---|---|
| `gemini-3.1-flash-lite-preview` | 500 | ✅ Best for learning — use this |
| `gemini-2.5-flash` | 20 | Use sparingly |
| `gemini-2.5-pro` | 0 | Paid only |

---

## What You Learned

```
✅ What MCP is — pre-built tool servers
✅ GitHub MCP — 26 tools, zero API code
✅ LangGraph agent — modern replacement for AgentExecutor
✅ Plain English GitHub operations
✅ Full GitHub lifecycle — repo to issue management
```

---
