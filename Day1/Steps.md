# DevOps AI Agent — Day 1
> Build your first AI agent that audits AWS infrastructure using Gemini + LangChain on AWS CloudShell.

---

## Prerequisites
- AWS account with CloudShell access
- Google account (for Gemini API key)
- Basic Python knowledge

---

## Step 1 — Get Gemini API Key
1. Go to [aistudio.google.com](https://aistudio.google.com)
2. Sign in with Google
3. Click **Get API Key** → **Create API key**
4. Copy the key

---

## Step 2 — Open AWS CloudShell
```
AWS Console → Click >_ icon (top right) → Wait for it to load
```

---

## Step 3 — Create Project
```bash
mkdir devops-agent && cd devops-agent
python3 -m venv venv
source venv/bin/activate
```

---

## Step 4 — Install Packages
```bash
pip install langchain langchain-google-genai langchain-community python-dotenv rich
```

---

## Step 5 — Create .env File
```bash
cat > .env << EOF
GOOGLE_API_KEY=paste_your_key_here
EOF
```

```bash
# Protect your key
cat > .gitignore << EOF
.env
venv/
__pycache__/
EOF
```

---

## Step 6 — Test Gemini Connection
```bash
cat > test.py << 'EOF'
from dotenv import load_dotenv
from langchain_google_genai import ChatGoogleGenerativeAI
import os
load_dotenv()

key = os.getenv("GOOGLE_API_KEY")
print(f"✅ Key loaded: {key[:8]}...hidden")

llm = ChatGoogleGenerativeAI(model="gemini-2.5-flash", temperature=0)
response = llm.invoke("Say hello in one word")
print("✅ Gemini works:", response.content)
EOF

python3 test.py
```

**Expected output:**
```
✅ Key loaded: AIzaSyXX...hidden
✅ Gemini works: Hello
```

---

## Step 7 — Create tools.py
```bash
cat > tools.py << 'EOF'
import subprocess
from langchain.tools import tool

def run_aws(cmd):
    """Run AWS CLI and filter CloudShell gRPC noise"""
    result = subprocess.run(cmd, capture_output=True, text=True)

    def clean(text):
        if not text:
            return ""
        return "\n".join([
            line for line in text.splitlines()
            if "ev_epoll1_linux" not in line
            and "epoll_wait" not in line
            and "Bad file descriptor" not in line
            and "event_engine" not in line
            and "Epoll1Poller" not in line
        ]).strip()

    return clean(result.stdout) or clean(result.stderr) or "No results found"

@tool
def get_s3_buckets(placeholder: str = "list") -> str:
    """List all S3 buckets in the AWS account"""
    return run_aws(["aws", "s3", "ls"])

@tool
def get_ec2_instances(placeholder: str = "list") -> str:
    """List all EC2 instances with their state and type"""
    return run_aws([
        "aws", "ec2", "describe-instances",
        "--query", "Reservations[].Instances[].{ID:InstanceId,State:State.Name,Type:InstanceType}",
        "--output", "table"
    ])

@tool
def get_vpcs(placeholder: str = "list") -> str:
    """List all VPCs in the AWS account"""
    return run_aws([
        "aws", "ec2", "describe-vpcs",
        "--query", "Vpcs[].{ID:VpcId,CIDR:CidrBlock,Default:IsDefault}",
        "--output", "table"
    ])

@tool
def get_iam_users(placeholder: str = "list") -> str:
    """List all IAM users in the AWS account"""
    return run_aws([
        "aws", "iam", "list-users",
        "--query", "Users[].{User:UserName,Created:CreateDate}",
        "--output", "table"
    ])

@tool
def get_security_groups(placeholder: str = "list") -> str:
    """List all security groups in the AWS account"""
    return run_aws([
        "aws", "ec2", "describe-security-groups",
        "--query", "SecurityGroups[].{ID:GroupId,Name:GroupName,VPC:VpcId}",
        "--output", "table"
    ])

@tool
def get_cloudwatch_alarms(placeholder: str = "list") -> str:
    """List all CloudWatch alarms and their states"""
    return run_aws([
        "aws", "cloudwatch", "describe-alarms",
        "--query", "MetricAlarms[].{Name:AlarmName,State:StateValue,Metric:MetricName}",
        "--output", "table"
    ])
EOF
```

---

## Step 8 — Create agent.py
```bash
cat > agent.py << 'EOF'
from dotenv import load_dotenv
from langchain_google_genai import ChatGoogleGenerativeAI
from langchain.agents import AgentExecutor, create_react_agent
from langchain import hub
from tools import (
    get_s3_buckets,
    get_ec2_instances,
    get_vpcs,
    get_iam_users,
    get_security_groups,
    get_cloudwatch_alarms
)

load_dotenv()

tools = [
    get_s3_buckets,
    get_ec2_instances,
    get_vpcs,
    get_iam_users,
    get_security_groups,
    get_cloudwatch_alarms
]

llm = ChatGoogleGenerativeAI(
    model="gemini-2.5-flash",
    temperature=0,
    max_retries=3,
    request_timeout=60
)

prompt = hub.pull("hwchase17/react")

agent = create_react_agent(llm, tools, prompt)
agent_executor = AgentExecutor(
    agent=agent,
    tools=tools,
    verbose=True,
    max_iterations=8,
    handle_parsing_errors=True
)

result = agent_executor.invoke({
    "input": """
    You are a senior DevOps assistant auditing an AWS account.
    Do the following steps one by one:
    1. List all S3 buckets
    2. List all EC2 instances and their states
    3. List all VPCs
    4. Check CloudWatch alarms and their states
    5. Give me a clean infrastructure summary with any issues or warnings you notice
    """
})

print("\n" + "="*50)
print("✅ INFRASTRUCTURE SUMMARY")
print("="*50)
print(result["output"])
EOF
```

---

## Step 9 — Run the Agent
```bash
python3 agent.py
```

You will see the agent thinking out loud:
```
> Entering new AgentExecutor chain...
Thought: I need to check S3 buckets first
Action: get_s3_buckets
Observation: No S3 buckets found
Thought: Now check EC2 instances...
...
✅ INFRASTRUCTURE SUMMARY
==================================================
Found X S3 buckets, Y EC2 instances...
```

---

## Project Structure
```
devops-agent/
├── .env          ← API key (never commit)
├── .gitignore    ← protects .env
├── test.py       ← verify Gemini connection
├── tools.py      ← agent's hands (AWS actions)
├── agent.py      ← agent's brain (reasoning loop)
└── venv/         ← isolated packages
```

---

## Troubleshooting

| Error | Fix |
|---|---|
| `ModuleNotFoundError` | Run `source venv/bin/activate` |
| `API key invalid` | Check key in `.env` file |
| `Rate limit hit` | Wait 60 seconds — free tier is 10 RPM |
| `Unable to locate credentials` | Run `aws sts get-caller-identity` to verify CloudShell AWS access |
| `model not found` | Use `gemini-2.5-flash` — verified working model |

---

## How It Works

```
You give a goal → Agent thinks (Gemini) → Picks a tool → Runs it
→ Reads output → Thinks again → Picks next tool → ... → Final Answer
```

This is called the **ReAct loop** — the foundation of all AI agents.

---
