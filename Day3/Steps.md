# 🚀 Episode 3 — AI CI/CD with GitHub Actions & MCP Agent

> Build, debug, fix, and enhance CI/CD pipelines using plain English.

---

## 🧠 What You'll Build

An AI DevOps agent that can:

* Create GitHub Actions pipelines
* Trigger workflows
* Debug failures
* Fix CI/CD issues automatically
* Merge production-ready code
* Build Docker images

---

## ⚙️ How It Works

```
You type:   "create CI pipeline"
Agent:      creates GitHub Actions workflow

You type:   "pipeline failed fix it"
Agent:      reads logs → fixes → updates workflow

Result:     working CI/CD pipeline 🚀
```

---

## 📌 Prerequisites

* Completed Day 2 setup
* AWS CloudShell
* Gemini API Key
* GitHub Personal Access Token

---

# ⚙️ Step 1 — Create Project Folder

```bash
mkdir day3 && cd day3
```

---

# 🐍 Step 2 — Create Virtual Environment

```bash
python3.11 -m venv venv
source venv/bin/activate
python3 --version
```

---

# 🔄 Step 3 — Upgrade pip

```bash
python3 -m pip install --upgrade pip
```

---

# 📦 Step 4 — Create requirements.txt

```bash
cat > requirements.txt << 'EOF'
langchain
langchain-google-genai
langchain-mcp-adapters
langgraph
python-dotenv
rich
requests
EOF
```

---

# 📥 Step 5 — Install Dependencies

```bash
pip install -r requirements.txt
```

---

# 🔐 Step 6 — Copy Environment Variables

```bash
cp ~/day2/.env .env
```

```bash
cat >> .env << EOF
GOOGLE_API_KEY=key
GITHUB_PERSONAL_ACCESS_TOKEN=token
GITHUB_USERNAME=sandesh2026
EOF
```
---

# ➕ Step 7 — Add Repository Name

```bash
cat >> .env << 'EOF'
GITHUB_REPO=ai-cicd-agent-day3
EOF
```

---

# 🧪 Step 8 — Create test.py

```bash
cat > test.py << 'EOF'
import asyncio
import os
from dotenv import load_dotenv
from langchain_google_genai import ChatGoogleGenerativeAI
from langchain_mcp_adapters.client import MultiServerMCPClient

load_dotenv()

print("\n🔍 Checking setup...\n")

llm = ChatGoogleGenerativeAI(
    model="gemini-3.1-flash-lite-preview",
    temperature=0
)

print("✅ Gemini connected")

async def check():
    client = MultiServerMCPClient(
        {
            "github": {
                "command": "npx",
                "args": ["-y", "@modelcontextprotocol/server-github"],
                "env": {
                    "GITHUB_PERSONAL_ACCESS_TOKEN": os.getenv("GITHUB_PERSONAL_ACCESS_TOKEN")
                },
                "transport": "stdio"
            }
        }
    )

    tools = await client.get_tools()
    print(f"✅ MCP Connected — {len(tools)} tools")

asyncio.run(check())

print("\n🚀 Ready for Day 3\n")
EOF
```

---

# ▶️ Step 9 — Run Test

```bash
python3 test.py
```

---

# 🤖 Step 10 — Create CI/CD Agent

```bash
cat > cicd_agent.py << 'EOF'
import asyncio
import os
from dotenv import load_dotenv
from langchain_google_genai import ChatGoogleGenerativeAI
from langchain_mcp_adapters.client import MultiServerMCPClient
from langgraph.prebuilt import create_react_agent

load_dotenv()

TOKEN = os.getenv("GITHUB_PERSONAL_ACCESS_TOKEN")
USERNAME = os.getenv("GITHUB_USERNAME")
REPO = os.getenv("GITHUB_REPO")


async def run(task: str):
    print("\n🔗 Connecting to GitHub MCP...\n")

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

    print(f"✅ Connected — {len(tools)} tools available\n")

    print("🛠 Sample Tools:")
    for t in tools[:5]:
        print(" -", t.name)
    print("\n" + "="*50)

    llm = ChatGoogleGenerativeAI(
        model="gemini-3.1-flash-lite-preview",
        temperature=0
    )

    agent = create_react_agent(llm, tools)

    print("\n🤖 Running your task...\n")

    result = await agent.ainvoke({
        "messages": [
            {
                "role": "user",
                "content": f"""
You are a senior DevOps engineer specializing in CI/CD using GitHub Actions.

Rules:
- Never commit directly to main branch
- Always create feature branches
- Always create pull requests
- Always fix CI/CD issues automatically
- Prefer taking actions over explaining

GitHub username : {USERNAME}
GitHub repo     : {REPO}

Task: {task}
"""
            }
        ]
    })

    final = result["messages"][-1].content

    print("\n" + "="*60)
    print("✅ RESULT")
    print("="*60)
    print(final)
    print("\n")


def get_input():
    print("\n🤖 AI CI/CD Agent — Day 3")
    print("="*60)
    print(f"👤 User : {USERNAME}")
    print(f"📦 Repo : {REPO}")
    print("="*60)

    print("\n👉 Enter your DevOps task below")
    print("👉 Type DONE on a new line to execute\n")

    lines = []
    while True:
        try:
            line = input("task >> ")
        except KeyboardInterrupt:
            print("\n❌ Interrupted. Exiting...")
            exit(0)

        if line.strip().upper() == "DONE":
            break

        lines.append(line)

    return "\n".join(lines)


if __name__ == "__main__":
    task = get_input()

    if not task.strip():
        print("⚠️ No task provided. Exiting.")
        exit(0)

    print("\n🚀 Executing task...\n")

    try:
        asyncio.run(run(task))
    except Exception as e:
        print("\n❌ Error occurred:")
        print(str(e))
EOF
```

---

# ▶️ Step 11 — Run Agent

```bash
python3 cicd_agent.py
```

---

# 🔥 Step 12 — Full CI/CD Flow

---

## Task 1 — Create Repository

```
Create a new public repository called ai-cicd-agent-day3

Description: "AI DevOps Day 3 - CI/CD Agent with GitHub MCP"

DONE
```

---

## Task 2 — Create README

```
Create a README.md file in ai-cicd-agent-day3 repo

Content:
# AI CI/CD Agent - Day 3

This project demonstrates AI-powered CI/CD using GitHub MCP.

DONE
```

---

## Task 3 — Create Branch

```
Create a new branch called feature/day3-cicd
from main in ai-cicd-agent-day3 repo

DONE
```

---

## Task 4 — Create Project Files

```
Create file day3/requirements.txt in feature/day3-cicd branch

Content:
pytest
requests

DONE
```

```
Create file day3/app.py in feature/day3-cicd branch

Content:
def add(a, b):
    return a + b

DONE
```

```
Create file day3/test_app.py in feature/day3-cicd branch

Content:
from app import add

def test_add():
    assert add(1, 2) == 3

DONE
```

---

## Task 5 — Create CI Pipeline (Intentional Mistake)

```
Create a GitHub Actions workflow for Python project feature/day3-cicd branch. branch already exists create workflow

Requirements:
- Install dependencies
- Run pytest
- Use caching

IMPORTANT:
Assume requirements.txt is in root

DONE
```

---

## Task 6 — Create Pull Request

```
Create a pull request

title: "feat: add CI/CD pipeline"
from feature/day3-cicd into main

DONE
```

---

## Task 7 — Observe Failure

Pipeline fails due to:

* incorrect file path
* test issues

---

## Task 8 — Fix Pipeline

```
The GitHub Actions pipeline failed in the repository ai-cicd-agent-day3.

Context:
- Repository: ai-cicd-agent-day3
- Branch: feature/day3-cicd
- Workflow file: .github/workflows/python-ci.yml
- The file exists in the feature/day3-cicd branch

Problem:
- The workflow is using incorrect dependency path:
  pip install -r requirements.txt
- But requirements.txt is located at:
  day3/requirements.txt
- Tests are also not running from the correct directory

Tasks:
1. Update the existing workflow file:
   .github/workflows/python-ci.yml
2. Modify dependency installation to:
   pip install -r day3/requirements.txt
3. Modify test step to:
   cd day3 && pytest
4. Commit changes explicitly to:
   feature/day3-cicd branch

Important:
- Do NOT create a new workflow file
- Do NOT modify main branch
- Only update the existing file in feature/day3-cicd

DONE
```

---

## Task 9 — Fix Test Issues

```
Tests are not detected or failing.

Fix:
- ensure pytest naming (test_*.py)
- fix imports if needed

DONE
```

---

## Task 10 — Verify Pipeline

```
Check the latest GitHub Actions workflow run in the repository ai-cicd-agent-day3.

Context:
- Repository: ai-cicd-agent-day3
- Branch: feature/day3-cicd

Tasks:
1. Find the latest workflow run triggered from this branch
2. Summarize the status (success / failed / pending)
3. If failed, explain the reason briefly

DONE
```

---

## Task 11 — Merge Pull Request

```
Merge the first open pull request

DONE
```

---

# 🐳 Step 13 — Add Docker Build

```
Enhance the existing GitHub Actions workflow in ai-cicd-agent-day3 repo to include Docker build.

Requirements:
- Build a Docker image for the project
- Use best practices
- Tag image as ai-cicd-agent-day3:latest
- Keep existing steps (install + pytest)

Update the workflow in main branch

DONE
```

---

## Step 14 — Create Dockerfile (if needed)

```
Create a Dockerfile in ai-cicd-agent-day3 repo

Requirements:
- Use python:3.11-slim
- Set working directory
- Copy day3 folder
- Install dependencies from day3/requirements.txt

DONE
```

---

## Step 15 — Trigger Pipeline Again

```
Make a small change in README.md and commit to main

DONE
```

---

## Step 16 — Verify Docker Build

```
Check latest workflow run status and confirm Docker build succeeded

DONE
```

---

# 📂 Project Structure

```
day3/
├── .env
├── requirements.txt
├── test.py
├── cicd_agent.py
└── venv/
```

---

# 🧠 What You Learned

```
✅ CI/CD with AI agents  
✅ GitHub Actions automation  
✅ Debugging pipelines using AI  
✅ Fixing real-world CI/CD issues  
✅ Docker integration in CI/CD  
```

---

## 🧠 DevOps Role Evolution

| Old Role        | New Role            |
|-----------------|---------------------|
| Typing commands | Designing systems   |
| Writing YAML    | Reviewing AI output |
| Fixing manually | Guiding AI to fix   |
| Doing work      | Orchestrating work  |

---
