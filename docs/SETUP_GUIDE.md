# HeliOS Studio Setup Guide

Step-by-step instructions to implement the HeliOS Studio stack.

## Prerequisites

- Proxmox homelab with at least one GPU (T600, RTX 4000 Quadro, or similar)
- GitHub account
- Basic familiarity with Docker, Linux, and Git
- Windows/Linux/Mac workstation

---

## Phase 1: Core Tools (Weekend 1)

### 1.1 Install Cursor

```bash
# Download from https://cursor.sh
# Or via package manager:
brew install --cask cursor  # macOS
```

**Configuration**:
1. Sign in with GitHub
2. Settings → AI → Add API keys:
   - Claude (Anthropic)
   - GPT (OpenAI)
   - Gemini (Google)
3. Enable Composer for multi-file edits
4. Set preferred model per task type

### 1.2 Install Claude Desktop

```bash
# Download from https://claude.ai/download
```

**Enable MCP**:
1. Open Claude Desktop settings
2. Enable "Experimental Features"
3. Configure MCP servers (see Phase 2)

### 1.3 Install Ollama (Workstation)

```bash
# Linux
curl -fsSL https://ollama.com/install.sh | sh

# macOS
brew install ollama

# Windows (WSL or native)
winget install Ollama.Ollama
```

**Pull initial models**:
```bash
ollama pull llama3        # 8B general model
ollama pull phi3          # 3.8B fast utility model
ollama pull qwen3         # 7B coding/reasoning
```

**Test**:
```bash
ollama run llama3 "Write a hello world in Python"
```

**Configure API** (optional, for remote access):
```bash
# Edit ~/.ollama/config
export OLLAMA_HOST=0.0.0.0:11434
```

### 1.4 Install Ollama (Homelab VM)

**Create VM on Proxmox**:
- Name: `ollama-server`
- OS: Ubuntu 22.04 LTS
- CPU: 8 cores
- RAM: 32GB
- GPU: Passthrough T600 or RTX 4000

**Install**:
```bash
ssh ollama-server
curl -fsSL https://ollama.com/install.sh | sh

# Install NVIDIA drivers if using GPU
sudo apt install nvidia-driver-535

# Verify GPU
nvidia-smi

# Pull larger models
ollama pull qwen3:30b    # 30B parameter model
```

**Expose API**:
```bash
sudo systemctl edit ollama

# Add:
[Service]
Environment="OLLAMA_HOST=0.0.0.0:11434"

sudo systemctl restart ollama
```

---

## Phase 2: Automation and MCP (Weekend 2)

### 2.1 Deploy n8n

**On Proxmox VM** (`automation-hub`):

```bash
# Create docker-compose.yml
version: '3.8'
services:
  n8n:
    image: n8nio/n8n:latest
    restart: always
    ports:
      - "5678:5678"
    environment:
      - N8N_BASIC_AUTH_ACTIVE=true
      - N8N_BASIC_AUTH_USER=admin
      - N8N_BASIC_AUTH_PASSWORD=<strong-password>
      - N8N_HOST=n8n.yourdomain.com
      - WEBHOOK_URL=https://n8n.yourdomain.com/
    volumes:
      - n8n_data:/home/node/.n8n
      - /var/run/docker.sock:/var/run/docker.sock

volumes:
  n8n_data:
```

```bash
docker-compose up -d
```

**Access**: http://your-vm-ip:5678

**First workflow**: Create a test workflow that logs a webhook payload.

### 2.2 Setup Filesystem MCP Server

**Install MCP tools**:
```bash
npm install -g @modelcontextprotocol/server-filesystem
```

**Configure Claude Desktop** (`~/Library/Application Support/Claude/claude_desktop_config.json` on macOS):

```json
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "/path/to/workspace"]
    }
  }
}
```

**Create workspace directory**:
```bash
mkdir -p ~/workspace/helios-studio
cd ~/workspace/helios-studio
```

**Test**: In Claude Desktop, ask "List files in my workspace"

### 2.3 Setup GitHub MCP Server

```bash
npm install -g @modelcontextprotocol/server-github
```

**Create GitHub fine-grained token**:
1. GitHub → Settings → Developer settings → Personal access tokens → Fine-grained tokens
2. Token name: `helios-studio-mcp`
3. Repository access: Select "HeliOS-Studio" and other studio repos
4. Permissions:
   - Contents: Read and write
   - Issues: Read and write
   - Pull requests: Read and write
   - Metadata: Read-only

**Configure Claude Desktop**:

```json
{
  "mcpServers": {
    "filesystem": { ... },
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_TOKEN": "ghp_your_token_here"
      }
    }
  }
}
```

**Test**: In Claude Desktop, ask "List issues in zebadee2kk/HeliOS-Studio"

### 2.4 Setup n8n MCP Server (Custom)

**Create simple Express server** (`~/workspace/mcp-servers/n8n/server.js`):

```javascript
const express = require('express');
const axios = require('axios');

const app = express();
app.use(express.json());

const N8N_URL = process.env.N8N_URL || 'http://localhost:5678';
const N8N_API_KEY = process.env.N8N_API_KEY;

app.post('/trigger', async (req, res) => {
  const { workflow, payload } = req.body;
  try {
    const result = await axios.post(
      `${N8N_URL}/webhook/${workflow}`,
      payload
    );
    res.json({ success: true, data: result.data });
  } catch (error) {
    res.status(500).json({ success: false, error: error.message });
  }
});

app.listen(3000, () => console.log('n8n MCP server on port 3000'));
```

**Run**:
```bash
node server.js
```

---

## Phase 3: Knowledge Pipeline (Weekend 3)

### 3.1 Setup NotebookLM

1. Go to https://notebooklm.google.com
2. Sign in with a dedicated Google account (studio identity, not personal)
3. Create notebooks:
   - `HeliOS-Opportunities-Cybersec`
   - `HeliOS-Opportunities-Devtools`
   - `HeliOS-Tech-Research`
   - `HeliOS-Experiments`

### 3.2 Create GitHub Knowledge Repos

```bash
gh repo create kb-opportunities-cybersec --private
gh repo create kb-opportunities-devtools --private
gh repo create kb-tech-research --private
gh repo create kb-feeds --private
```

**Add structure**:
```bash
cd kb-opportunities-cybersec
mkdir -p {research,summaries,opportunities}
echo "# Cybersecurity Opportunities" > README.md
git add -A && git commit -m "Initial structure" && git push
```

### 3.3 Setup Manual Research Flow

**Process**:
1. Perplexity: Research a topic, save to Collection
2. NotebookLM: Create notebook, add Perplexity links as sources
3. NotebookLM: Run Deep Research
4. NotebookLM: Export report as PDF/Doc to Google Drive
5. Download and commit to appropriate GitHub kb-repo

**Example**:
```bash
cd kb-opportunities-cybersec/research
cp ~/Downloads/mcp-security-gaps-report.pdf .
git add mcp-security-gaps-report.pdf
git commit -m "Add MCP security gaps research from NotebookLM"
git push
```

### 3.4 Automate Export (Future)

**n8n workflow** (to be built):
1. Trigger: Google Drive file created in "NotebookLM Exports" folder
2. Action: Download file
3. Action: Commit to appropriate GitHub repo based on filename pattern
4. Action: Notify via email/Slack

---

## Phase 4: R&D Workflows (Weekend 4)

### 4.1 Create Studio Inbox Repo

```bash
gh repo create studio-inbox --private
cd studio-inbox
```

**Create issue template** (`.github/ISSUE_TEMPLATE/idea.md`):

```markdown
---
name: Project Idea
about: New project or opportunity to explore
title: '[IDEA] '
labels: 'idea'
---

## Problem
What problem does this solve?

## Users
Who needs this?

## Constraints
Time, budget, technical limitations?

## Success Criteria
How do we know it works?

## Research
Link to NotebookLM notebook or GitHub kb-repo
```

### 4.2 Create Claude Prompt Template

**Save as** `~/workspace/prompts/idea-to-plan.md`:

```markdown
# Idea to Plan

Given this issue and research notebook, produce:

1. **Architecture Decision Records (ADRs)**
   - Key technical decisions
   - Trade-offs considered
   - Rationale

2. **Architecture Diagram** (Mermaid)
   - System components
   - Data flow
   - Integration points

3. **90-Day Roadmap**
   - Week 1-4: MVP
   - Week 5-8: Core features
   - Week 9-12: Polish and launch

4. **Threat Model** (for security-related projects)
   - Attack surface
   - Mitigations
   - Security controls

5. **GitHub Issues**
   - Milestone issues with acceptance criteria
   - Labeled by priority and type

Format output as markdown files ready to commit.
```

### 4.3 Setup Auto-Planning Workflow (n8n)

**Workflow**:
1. Trigger: GitHub webhook on new issue with label `auto-plan`
2. Action: Fetch issue body and linked research
3. Action: Call Claude API with idea-to-plan prompt
4. Action: Parse response, create GitHub issues for roadmap
5. Action: Comment on original issue with summary and links

---

## Phase 5: Experiments (Weekend 5)

### 5.1 Create Project Template

```bash
gh repo create project-template --public --template
cd project-template
```

**Add standard files**:

`docker-compose.yml`:
```yaml
version: '3.8'
services:
  app:
    build: .
    ports:
      - "8000:8000"
    environment:
      - DATABASE_URL=postgresql://user:pass@db:5432/app
    depends_on:
      - db
  db:
    image: postgres:16
    environment:
      - POSTGRES_USER=user
      - POSTGRES_PASSWORD=pass
      - POSTGRES_DB=app
    volumes:
      - pgdata:/var/lib/postgresql/data
volumes:
  pgdata:
```

`.github/workflows/ci.yml`:
```yaml
name: CI
on: [push, pull_request]
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-python@v4
        with:
          python-version: '3.11'
      - run: pip install -r requirements.txt
      - run: pytest
      - run: ruff check .
```

### 5.2 Setup Experiment Runner (n8n)

**Workflow**:
1. Trigger: Cron (nightly)
2. Action: Call project `/experiment` endpoint with test scenarios
3. Action: Aggregate metrics (latency, accuracy, cost)
4. Action: Write markdown report to repo `experiments/YYYY-MM-DD.md`
5. Action: Update NotebookLM experiment notebook (via Drive API)

---

## Verification Checklist

- [ ] Cursor installed and configured with API keys
- [ ] Claude Desktop installed with MCP enabled
- [ ] Ollama running on workstation with 3+ models
- [ ] Ollama running on homelab VM with GPU
- [ ] n8n deployed and accessible
- [ ] Filesystem MCP server working
- [ ] GitHub MCP server working
- [ ] NotebookLM account created with 4 initial notebooks
- [ ] GitHub knowledge repos created (4 repos)
- [ ] Studio inbox repo with issue template
- [ ] Project template repo with CI/CD
- [ ] First test workflow in n8n

---

## Troubleshooting

### Ollama GPU not detected

```bash
# Verify NVIDIA drivers
nvidia-smi

# Reinstall if needed
sudo apt purge nvidia-*
sudo apt install nvidia-driver-535
sudo reboot
```

### MCP servers not connecting

1. Check Claude Desktop logs: `Help → Show Logs`
2. Verify MCP config syntax (valid JSON)
3. Test MCP server manually:

```bash
npx -y @modelcontextprotocol/server-filesystem /tmp
```

### n8n webhooks not receiving

1. Check n8n logs: `docker logs n8n`
2. Verify webhook URL is publicly accessible
3. Test with curl:

```bash
curl -X POST https://your-n8n.com/webhook/test \
  -H "Content-Type: application/json" \
  -d '{"test": true}'
```

---

## Next Steps

See [WORKFLOWS.md](WORKFLOWS.md) for end-to-end examples.
