# HeliOS Studio Workflows

End-to-end examples showing how tools integrate.

---

## Workflow 1: Opportunity → Research → Architecture

### Goal
Identify a market opportunity, research it thoroughly, and design a solution.

### Steps

**1. Discovery (Perplexity)**

Query: `"emerging security gaps in MCP-based AI agent systems 2026"`

- Read results
- Save promising answers to Collection "MCP Security"
- Export 5-10 key links

**2. Deep Research (NotebookLM)**

- Create notebook: "MCP Security Gaps 2026"
- Add Perplexity links as sources (paste URLs)
- Add relevant GitHub repos, PDFs, YouTube videos
- Run **Deep Research**: "Comprehensive analysis of MCP security vulnerabilities and mitigation strategies"
- Wait 3-5 minutes for report generation
- Review:
  - Timeline of MCP security incidents
  - Mind map of attack vectors
  - Summary of existing tools and gaps

**3. Export and Commit (Manual → Automated)**

- NotebookLM: Export Deep Research report to Google Drive as PDF
- Download to local machine
- Commit to GitHub:

```bash
cd kb-opportunities-cybersec/research
cp ~/Downloads/mcp-security-gaps-2026.pdf .
git add mcp-security-gaps-2026.pdf
git commit -m "Deep research: MCP security gaps and opportunities"
git push
```

**4. Architecture (Claude + Claude Code)**

- Open Claude Desktop
- Attach: 
  - GitHub repo (`kb-opportunities-cybersec`)
  - PDF report via filesystem MCP
- Prompt:

```
Analyze this research on MCP security gaps.

Design a minimal viable product that addresses the top 3 gaps:
1. Over-privileged MCP servers
2. Missing audit logging
3. Token passthrough vulnerabilities

Produce:
- Architecture diagram (Mermaid)
- ADRs for key decisions
- Threat model
- 90-day build plan
```

- Claude generates:
  - `architecture.md` with Mermaid diagram
  - `adr/001-mcp-sandboxing.md`
  - `adr/002-audit-logging.md`
  - `threat-model.md`
  - `roadmap.md`

**5. Save Architecture**

- Use Claude Code filesystem MCP to write files to workspace
- Or copy/paste into new GitHub repo:

```bash
gh repo create mcp-security-scanner --private
cd mcp-security-scanner
mkdir -p docs/{architecture,adr,security}
# Paste Claude outputs
git add -A && git commit -m "Initial architecture" && git push
```

---

## Workflow 2: Architecture → Prototype → Deploy

### Goal
Build and deploy a working prototype from architecture docs.

### Steps

**1. Scaffold Repo (Claude Code)**

- Open Claude Code
- Attach GitHub repo `mcp-security-scanner`
- Prompt:

```
Based on the architecture in docs/, scaffold a minimal FastAPI application:

- Project structure following best practices
- docker-compose.yml with app + postgres + redis
- GitHub Actions CI/CD workflow
- Pre-commit hooks for linting and security scanning
- README with setup instructions
```

- Claude Code creates:
  - `src/main.py` (FastAPI skeleton)
  - `docker-compose.yml`
  - `.github/workflows/ci.yml`
  - `requirements.txt`
  - `pyproject.toml`
  - `README.md`

**2. Implement Features (Cursor)**

- Open repo in Cursor
- Use Composer for multi-file changes:
  - Implement MCP server scanner
  - Add audit logging
  - Write tests

**3. Local Testing**

```bash
docker-compose up
curl http://localhost:8000/health
```

**4. Deploy to Homelab**

- Push to GitHub
- GitHub Actions builds Docker image
- Self-hosted runner on Proxmox pulls and deploys

**5. Run Experiments (n8n)**

- n8n workflow triggers nightly:
  - Calls `/scan` endpoint with test MCP configs
  - Collects metrics (vulnerabilities found, false positives)
  - Writes report to `experiments/YYYY-MM-DD.md`
  - Updates NotebookLM experiment log

---

## Workflow 3: Automated Research Capture

### Goal
Automatically capture and organize new research from feeds.

### n8n Workflow

**Trigger**: Cron (daily at 8am)

**Nodes**:

1. **RSS Node**: Fetch from security feeds
   - https://threatpost.com/feed/
   - https://www.bleepingcomputer.com/feed/
   - GitHub Security Advisories API

2. **Filter Node**: Only items matching keywords
   - "MCP", "AI security", "prompt injection", "LLM vulnerability"

3. **Transform Node**: Extract title, link, summary

4. **GitHub Node**: Append to `kb-feeds/daily/YYYY-MM-DD.md`

5. **NotebookLM Node** (future, via API): Add links to "Daily Security Feed" notebook

6. **Notification**: Email digest of new items

**Result**: Wake up to curated research links ready for review.

---

## Workflow 4: Automatic Project Creation

### Goal
Turn a GitHub issue into a new project repo automatically.

### n8n Workflow

**Trigger**: GitHub webhook on new issue in `studio-inbox` with label `new-project`

**Nodes**:

1. **Parse Issue**: Extract:
   - Title
   - Problem statement
   - Target stack (from issue body tags)
   - Research links

2. **Claude API Node**: Generate:
   - Project name (slug)
   - Initial README
   - Architecture outline

3. **GitHub Node**: Create new repo from template
   - Repo name: `{project-slug}`
   - Initialize from `project-template`
   - Add README and architecture docs

4. **GitHub Node**: Create initial issues
   - "Design with Claude" (link to research)
   - "Implement MVP"
   - "Setup CI/CD"

5. **Comment on Original Issue**: Link to new repo

**Result**: One-click project bootstrapping.

---

## Workflow 5: Weekly Lab Report

### Goal
Summarize all activity and suggest next steps.

### n8n Workflow

**Trigger**: Cron (Monday 9am)

**Nodes**:

1. **GitHub Node**: Fetch last week's activity
   - New issues
   - Merged PRs
   - Experiment results

2. **Ollama Node**: Summarize activity
   - Model: `llama3`
   - Prompt: "Summarize this week's R&D activity in 3 paragraphs"

3. **NotebookLM Node** (future): Update "Lab Journal" notebook

4. **Claude API Node**: Analyze and suggest
   - Prompt: "Based on this activity, what should I prioritize next week?"

5. **GitHub Node**: Create issue in `studio-inbox`
   - Title: "Weekly Lab Report - YYYY-MM-DD"
   - Body: Summary + recommendations

6. **Email Node**: Send report

**Result**: Automated weekly review and planning.

---
## Workflow 6: Experiment Pipeline

### Goal
Continuously test prototypes and log results.

### Per-Project n8n Workflow

**Trigger**: Cron (nightly at 2am)

**Nodes**:

1. **HTTP Request**: Call project `/experiment` endpoint
   - POST scenarios defined in repo `experiments/scenarios.json`

2. **Process Results**: Calculate metrics
   - Success rate
   - Latency (p50, p95, p99)
   - Error types
   - Cost (API calls, tokens)

3. **Write Report**: Generate markdown

```markdown
# Experiment Report - 2026-03-09

## Scenarios
- Scenario A: 95% success, 230ms p95
- Scenario B: 78% success, 890ms p95

## Issues
- Timeout on large inputs
- False positives in pattern X

## Recommendations
- Increase timeout to 5s
- Refine regex for pattern X
```

4. **GitHub Node**: Commit to `experiments/YYYY-MM-DD.md`

5. **Conditional**: If success rate < 90%, create GitHub issue

**Result**: Automated quality monitoring and regression detection.

---

## Workflow 7: MCP-Driven Multi-Repo Refactor

### Goal
Make a change across multiple repos simultaneously.

### Process

**Setup**: All studio repos follow same structure

**Use Case**: Update all projects to new security best practice

**Steps**:

1. **Claude Code with GitHub MCP**:

Prompt:
```
Across all repos in the HeliOS-Studio org:
1. Update docker-compose.yml to use Docker secrets instead of environment variables
2. Add .env.example file
3. Update README security section
4. Create PR for each repo with title "Security: Migrate to Docker secrets"
```

2. **Claude Code**:
   - Lists all repos via MCP
   - Clones each locally (via filesystem MCP)
   - Makes changes
   - Creates branch
   - Pushes and opens PR (via GitHub MCP)

3. **Review**: You review PRs, approve, and merge

**Result**: Bulk changes with AI assistance but human oversight.

---

## Workflow 8: Local LLM Integration

### Goal
Use local models for private/cheap tasks.

### Use Cases

**1. Log Summarization**

Daily n8n workflow:
- Fetch logs from all services
- Call local Ollama (Phi-3 Mini on CPU)
- Generate summary
- Post to GitHub wiki

**2. Code Review Assistant**

GitHub Action on PR:
```yaml
name: AI Code Review
on: pull_request
jobs:
  review:
    runs-on: self-hosted  # homelab runner
    steps:
      - uses: actions/checkout@v3
      - name: Review with local LLM
        run: |
          git diff origin/main...HEAD | \
          curl -X POST http://ollama-server:11434/v1/chat/completions \
            -H "Content-Type: application/json" \
            -d '{
              "model": "qwen3",
              "messages": [{
                "role": "user",
                "content": "Review this diff for security issues and bugs: $(cat)"
              }]
            }' | \
          jq -r '.choices[0].message.content' > review.md
      - name: Post comment
        uses: actions/github-script@v6
        with:
          script: |
            const review = require('fs').readFileSync('review.md', 'utf8');
            github.rest.issues.createComment({
              issue_number: context.issue.number,
              owner: context.repo.owner,
              repo: context.repo.repo,
              body: `## 🤖 Local AI Review\n\n${review}`
            });
```

**3. Documentation Generation**

Post-commit hook:
```bash
#!/bin/bash
# Generate API docs from code using local LLM

cat src/**/*.py | \
curl -X POST http://localhost:11434/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d @- <<EOF | jq -r '.choices[0].message.content' > API.md
{
  "model": "llama3",
  "messages": [{
    "role": "user",
    "content": "Generate API documentation in markdown from this Python code: $(cat)"
  }]
}
EOF

git add API.md
```

---

## Summary

All workflows follow the pattern:

1. **Trigger**: Manual, scheduled, or event-driven
2. **Data gathering**: From GitHub, web, or sensors
3. **AI processing**: Via Claude API, local Ollama, or MCP tools
4. **Action**: Commit to GitHub, create issues, deploy, or notify
5. **Logging**: All activity tracked in repos and NotebookLM

Key principle: **Human in the loop** for high-stakes decisions; automation for repetitive tasks.

---

## Next Steps

See [AUTOMATION.md](AUTOMATION.md) for detailed n8n workflow recipes.
