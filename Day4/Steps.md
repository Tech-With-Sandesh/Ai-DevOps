# 🚀 Episode 4 — AI DevOps Agent for Docker (From Scratch)
> Build, run, debug, manage, optimize, and clean Docker environments using AI

---

## 🧠 What You'll Build

In this episode, you will:

- Create a Python Flask app using AI  
- Generate a Dockerfile  
- Build Docker images  
- Run and manage containers  
- Debug real Docker issues  
- Tag and version images  
- Optimize Dockerfile  
- Clean up resources  
- Fix broken containers using AI  

---

## 🎯 Why This Matters

```
App → Dockerfile → Image → Container → Debug → Optimize → Version → Cleanup
```

👉 This is **real DevOps lifecycle**

---

## 📌 Prerequisites

- AWS CloudShell / Linux terminal  
- Python 3.11  
- Gemini API key  

---

# ⚙️ Step 1 — Setup Project

```bash
mkdir day4 && cd day4
python3.11 -m venv venv
source venv/bin/activate
pip install langchain-google-genai python-dotenv
```

---

# 🔐 Step 2 — Create .env

```bash
cat > .env << EOF
GOOGLE_API_KEY=your_key_here
EOF
```

---

# 🐳 Step 3 — Install Docker (CloudShell)

```bash
sudo yum install docker -y
sudo dockerd &
```

Verify:

```bash
docker --version
```

---

# 🤖 Step 4 — Create AI Agent

```bash
cat > docker_agent.py << 'EOF'
import subprocess
from dotenv import load_dotenv
from langchain_google_genai import ChatGoogleGenerativeAI

load_dotenv()

llm = ChatGoogleGenerativeAI(
    model="gemini-3.1-flash-lite-preview",
    temperature=0
)

def run_command(cmd):
    try:
        result = subprocess.run(cmd, shell=True, capture_output=True, text=True)
        return result.stdout + result.stderr
    except Exception as e:
        return str(e)

def get_multiline_input():
    print("\n👉 Enter your task (type DONE on new line to execute)\n")
    lines = []
    while True:
        line = input(">> ")
        if line.strip().upper().startswith("DONE"):
            break
        lines.append(line)
    return "\n".join(lines).strip()

def extract_command(response):
    content = response.content
    if isinstance(content, list):
        for item in content:
            if isinstance(item, dict) and "text" in item:
                return item["text"]
    if isinstance(content, dict) and "text" in content:
        return content["text"]
    return str(content)

def process_task(task):
    prompt = f"""
Convert the following DevOps task into shell commands.

Task:
{task}

Rules:
- Return ONLY commands
- No explanation
- No placeholders like <container_id>
- Use real commands
"""
    response = llm.invoke(prompt)
    commands = extract_command(response).strip()
    print("\n🧠 Commands:\n", commands)
    print("\n⚙️ Running...\n")
    print(run_command(commands))

def main():
    print("\n🤖 Docker AI Agent — Day 4\n")
    while True:
        task = get_multiline_input()
        if task.lower() == "exit":
            break
        process_task(task)

if __name__ == "__main__":
    main()
EOF
```

---

# ▶️ Step 5 — Run Agent

```bash
python3 docker_agent.py
```

---

# 🔥 Step 6 — Create Flask App

```text
Create a Python Flask app.

Requirements:
- file app.py
- return "Hello from AI DevOps Agent"
- run on port 5000
- install flask
- run app in background using &

DONE
```

---

# 🐳 Step 7 — Create Dockerfile

```text
Create a Dockerfile.

Context:
- app.py exists
- Flask app on port 5000

Requirements:
- python:3.11-slim
- copy files
- install flask
- expose 5000
- run app

DONE
```

---

# 🏗 Step 8 — Build Image

```text
Build Docker image.

Requirements:
- tag image as ai-devops-docker:latest
- show build output

DONE
```

---

# ▶️ Step 9 — Run Container

```text
Run Docker container.

Requirements:
- use image ai-devops-docker:latest
- map port 5001:5000
- run in background
- verify using docker ps

DONE
```

---

# 🔍 Step 10 — Check Logs

```text
Fetch container logs.

Requirements:
- use docker ps -q
- pass dynamically to docker logs
- show last 100 lines

DONE
```

---

# 🧪 Step 11 — Access Container

```text
Access container.

Requirements:
- use docker exec (no -it)
- run ls -la
- show output

DONE
```
```text
Access the running Docker container.

Requirements:
- First identify running container using docker ps
- Extract the container ID
- Use docker exec (no -it)
- Run ls -la inside container
- Show output

IMPORTANT:
- Do NOT use placeholders like container_name or <container_id>
- Use actual container ID dynamically
- Ensure command is executable

DONE
```
---

# 🔁 Step 12 — Restart Container

```text
Restart container.

Requirements:
- dynamically get container ID
- stop container
- start again
- verify using docker ps

DONE
```

---

# 🏷 Step 13 — Tag Image (Versioning)

```text
Tag the Docker image.

Requirements:
- tag ai-devops-docker:latest as ai-devops-docker:v1
- list images to verify

DONE
```

---

# 📦 Step 14 — Push Concept (Optional Explanation)

👉 Explain (no execution required):

```text
docker tag ai-devops-docker:v1 your-dockerhub-username/app:v1
docker push your-dockerhub-username/app:v1
```

---

# ⚡ Step 15 — Optimize Dockerfile

```text
Optimize Dockerfile.

Requirements:
- reduce image size
- use best practices
- improve build caching

DONE
```

---

# 🧹 Step 16 — Cleanup Environment

```text
Clean Docker environment.

Requirements:
- remove stopped containers
- remove unused images
- free disk space

DONE
```

---

# 📊 Step 17 — Inspect Container

```text
Inspect running container.

Requirements:
- use docker inspect
- show important details (IP, config)

DONE
```

---

# 📈 Step 18 — Monitor Resources

```text
Check container resource usage.

Requirements:
- use docker stats
- show CPU and memory usage

DONE
```

---

# Break & Fix (AI DevOps Agent with Docker)

# 🚨 Step 1 — Break the Application (Manual)

```bash
cat > app.py << 'EOF'
from flask import Flask

app = Flask(__name__)

@app.route("/")
def home():
    return unknown_variable   # ❌ intentional error

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000)
EOF
```

---

# 🏗 Step 2 — Rebuild Docker Image (Agent Prompt)

```text
Rebuild the Docker image after code change.

Requirements:
- Use docker build command
- Tag image as ai-devops-docker:latest
- Ensure build completes successfully

DONE
```

---

# ▶️ Step 3 — Run Container (Agent Prompt)

```text
Run the Docker container with updated image.

Requirements:
- Use image ai-devops-docker:latest
- Map port 5002:5000 (avoid conflict)
- Run container in background
- Verify using docker ps

IMPORTANT:
- Do NOT use placeholders
- Use actual values

DONE
```
```text
docker exec $(docker ps -q) python -c "import urllib.request; print(urllib.request.urlopen('http://localhost:5000').read())"
```


---

# ❌ Step 4 — Check Failure (Agent Prompt)

```text
Check the Docker container logs to identify any errors.

Steps:
1. Check if any container is running using docker ps
2. If no container is running:
   - use docker ps -a to find the latest container
3. Get the container ID dynamically
4. Fetch full logs using docker logs

IMPORTANT:
- Do NOT filter logs using grep
- Show complete logs output
- Do NOT use placeholders like <container_id>
- Use actual container ID dynamically

DONE
```

---

# 🔍 Step 5 — Identify Root Cause (Agent Prompt)

```text
Analyze the container logs.

Requirements:
- Identify the exact error
- Identify which file and line caused the issue
- Clearly explain root cause

DONE
```

---

# 🤖 Step 6 — Fix Code (Agent Prompt)

```text
Fix the issue in the Python application.

Context:
- Flask app inside app.py
- Error found in logs

Requirements:
- Correct the code
- Ensure no runtime errors
- Keep functionality same

DONE
```
```text
Fix the Python Flask application code.

Context:
- app.py contains a runtime error (unknown_variable)
- Application also had port conflict on 5000

Tasks:
1. Open app.py
2. Replace the incorrect line:
   return unknown_variable

3. Fix it to:
   return "Hello from AI DevOps Agent"

4. Update the application to run on a different port (use 5001 instead of 5000)

5. Save the file

6. Run the application

IMPORTANT:
- Modify the file using shell commands (sed, echo, or cat)
- Ensure code is valid and runnable
- Do NOT use port 5000
- Do NOT use placeholders
- Ensure application starts successfully

DONE
```
---

# 🏗 Step 7 — Rebuild Image Again (Agent Prompt)

```text
Rebuild the Docker image after fixing the code.

Requirements:
- Use docker build
- Tag image as ai-devops-docker:latest
- Ensure successful build

DONE
```

---

# ▶️ Step 8 — Restart Container (Agent Prompt)

```text
Restart the Docker container using the updated image.

Context:
- Docker image name: ai-devops-docker:latest
- Previous containers may still be running or stopped

Tasks:
1. Stop all running containers using:
   docker ps -q

2. Remove all existing containers using:
   docker ps -aq

3. Run a new container using:
   docker run -d -p 5003:5000 ai-devops-docker:latest

4. Verify the container is running using:
   docker ps

IMPORTANT:
- Do NOT use placeholders like <container_id> or <image_name>
- Use actual commands exactly as specified
- Ensure only one container is running
- Ensure port mapping 5003:5000 is used

DONE
```

---

# ✅ Step 9 — Verify Fix (Agent Prompt)

```text
Verify application is working.

Steps:
- Get container ID using docker ps -q
- Show logs using docker logs
- Trigger app using python request inside container
- Confirm no errors and valid response

IMPORTANT:
- No placeholders
- Use actual container ID

DONE
```

---


# 🎯 Final Output

```
App created ✅
Dockerfile created ✅
Image built ✅
Container running ✅
Logs checked ✅
Container accessed ✅
Restarted ✅
Image tagged ✅
Dockerfile optimized ✅
Resources cleaned ✅
Container monitored ✅
Issue fixed ✅
```

---

# 🧠 What You Learned

- Docker lifecycle (end-to-end)  
- Container debugging  
- Image versioning  
- Resource monitoring  
- AI limitations  
- Prompt engineering  
- Real DevOps workflows  


# 🔥 Role Transformation

| Old Role            | New Role (Day 4)        |
|--------------------|-------------------------|
| Writing commands   | Designing workflows     |
| Running Docker     | Guiding AI execution    |
| Debugging manually | Debugging with AI       |
| Fixing issues      | Orchestrating fixes     |
| Doing everything   | Reviewing + controlling |
