# HeliOS Studio Tool Stack

Recommended tools for each layer of the stack, with rationale and alternatives.

## Discovery Layer

### Primary: Perplexity AI

**Why**: Best-in-class web research with citations; Collections feature for organizing research themes.

**Use for**:
- Opportunity scanning ("emerging security gaps in MCP-based agents")
- Market research ("developer pain points in AI key management 2026")
- Competitive analysis
- Trend identification

**Alternatives**:
- ChatGPT with web search (good, but weaker citation and source quality)
- Bing Chat (free, but less organized)

---

## Knowledge Layer

### Primary: NotebookLM

**Why**: Purpose-built for research; handles diverse sources (PDFs, URLs, videos); generates timelines, mind maps, audio summaries, and comprehensive Deep Research reports.

**Use for**:
- Deep topic dives
- Literature reviews
- Experiment logs
- Long-term project knowledge base

**Features**:
- Deep Research (comprehensive multi-source analysis)
- Multiple export formats (summaries, timelines, mind maps, flashcards, audio, video)
- Support for 100+ sources per notebook
- PDF, URL, Google Doc, YouTube, image ingestion

**Alternatives**:
- Obsidian + local markdown (more control, less AI assistance)
- Notion AI (better collaboration, weaker research focus)

### Secondary: GitHub Knowledge Repos

**Why**: Version-controlled, searchable, integrates with rest of workflow.

**Structure**:
```
kb-opportunities-cybersec/
kb-opportunities-devtools/
kb-tech-rag-evals/
kb-tech-mcp-security/
kb-feeds/  # Raw RSS/API captures
```

---

## Reasoning Layer

### Primary: Claude (Anthropic)

**Why**: Strongest reasoning, longest context (200k tokens), excellent for architecture and system design.

**Use for**:
- System design
- Threat modeling
- ADRs and RFCs
- Architecture diagrams
- Trade-off analysis

**Access methods**:
- Web interface for quick iteration
- Desktop app with MCP support for tool integration

### Primary: Claude Code

**Why**: Repo-aware planning, large refactors, GitHub automation via MCP.

**Use for**:
- Codebase analysis ("What's the architecture?")
- Multi-file refactors
- Repo scaffolding
- GitHub automation (create repos, issues, PRs)

**MCP capabilities**:
- Filesystem access (sandboxed)
- GitHub operations
- Custom tool integration

### Secondary: Local LLMs (via Ollama)

**Why**: Private, offline, cost-free for suitable tasks.

**Models**:
- **Qwen3-7B / Llama 3-8B**: General reasoning and chat
- **DeepSeek / Qwen3 thinking variants**: Long reasoning chains
- **Phi-3 Mini**: Fast CPU-friendly utility tasks

**Use for**:
- Private/sensitive analysis
- Offline work
- Cheap batch processing

---

## Coding Layer

### Primary: Cursor

**Why**: AI-native, best-in-class repo awareness, multi-file edits, model flexibility.

**Features**:
- Full repo indexing (272k+ token context)
- Composer for multi-file edits
- Model selection per task (Claude, GPT, Gemini, custom)
- Tab completion
- AI-powered debugging

**Price**: ~$20/month

**Best for**: Feature work, refactors, new projects

### Secondary: VS Code + Extensions

**Why**: Existing workflows, extensions, corporate Copilot license.

**Extensions**:
- GitHub Copilot (OpenAI models)
- Cody (Sourcegraph, strong codebase search)
- Continue (open-source, multi-model)

**Best for**: Infrastructure scripting, specialized extensions

### Tertiary: Claude Code (CLI/outside IDE)

**Why**: Cross-repo automation, migration scripts.

**Best for**: Batch operations, repo setup, multi-repo changes

---

## Local AI Layer

### Primary: Ollama

**Why**: One-line install, 100+ models, OpenAI-compatible API, active development.

**Recommended models**:

| Model | Size | Use Case | Speed (est.) |
|-------|------|----------|-------------|
| Phi-3 Mini | 3.8B | Summarization, utility | 50+ tok/s (CPU) |
| Qwen3 | 4-7B | General chat, docs | 30-40 tok/s (GPU) |
| Llama 3 | 8B | Reasoning, coding | 20-30 tok/s (GPU) |
| Qwen3-Coder | 7B | Code understanding | 20-30 tok/s (GPU) |
| Qwen3 | 30B-A3B | Heavy coding, analysis | 10-15 tok/s (high-end GPU) |

**Installation**:
```bash
curl -fsSL https://ollama.com/install.sh | sh
ollama pull llama3
ollama pull qwen3
ollama pull phi3
```

**API**:
```bash
# OpenAI-compatible endpoint
curl http://localhost:11434/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{"model": "llama3", "messages": [{"role": "user", "content": "Hello"}]}'
```

**Alternatives**:
- LM Studio (GUI, easier for non-technical)
- llama.cpp (more control, harder setup)

---

## Automation Layer

### Primary: n8n (self-hosted)

**Why**: Visual workflow builder, self-hosted, AI nodes, 400+ integrations.

**Use for**:
- GitHub webhook → action workflows
- Scheduled research captures
- Experiment orchestration
- Data pipelines

**Deployment**:
```bash
docker run -d \
  --name n8n \
  -p 5678:5678 \
  -v n8n_data:/home/node/.n8n \
  n8nio/n8n
```

**Key features**:
- GitHub, GitLab, HTTP, webhook nodes
- AI nodes (OpenAI, local models)
- Database connectors
- Cron scheduling
- Error handling and retries

**Alternatives**:
- Zapier (easier, not self-hosted, more expensive)
- Airflow (more powerful, steeper learning curve)
- Make.com (similar to Zapier)

### Secondary: MCP Tools

**Why**: Standardized interfaces for Claude/Claude Code to call external tools.

**Key servers**:
- **Filesystem MCP**: Sandboxed file operations
- **GitHub MCP**: Repo, issue, PR management
- **n8n MCP**: Trigger workflows, query status
- **Ollama MCP**: Local inference

**Security**: Each server runs sandboxed with least-privilege access.

---

## Infrastructure Layer

### Version Control: GitHub

**Why**: Industry standard, excellent Actions CI/CD, free private repos, project management.

**Use for**:
- Source of truth
- Issue tracking
- Project boards
- CI/CD (Actions)
- Documentation (Pages)

### Containers: Docker + docker-compose

**Why**: Portable, reproducible, integrates with everything.

**Standard stack per project**:
```yaml
services:
  app:
    build: .
    ports:
      - "8000:8000"
  db:
    image: postgres:16
  redis:
    image: redis:7
```

### Virtualization: Proxmox

**Why**: Already in place, powerful, supports GPU passthrough.

**VM allocation**:
- AI Core: Ollama, vector DBs, orchestrator APIs
- Automation Hub: n8n, webhooks, monitoring
- Runners: Self-hosted GitHub Actions runners
- Prototypes: Project deployments

### Cloud: Minimal VPS

**Use for**: Public-facing endpoints only; prefer homelab for everything else.

**Providers**: Hetzner, Digital Ocean, Linode (cost-effective)

---

## Developer Environment Comparison

| Feature | VS Code + Copilot | Cursor | Windsurf | JetBrains + AI |
|---------|-------------------|--------|----------|----------------|
| AI integration | Extension-based | Native | Native | Native |
| Repo awareness | Limited | Excellent | Excellent | Excellent |
| Multi-file edits | Manual | Composer | Cascade | Junie |
| Model choice | OpenAI only | Multi-provider | Multi-provider | Multi-provider |
| Local model support | Via extensions | Yes | Yes | Yes |
| Price | $10/mo (Copilot) | $20/mo | $15/mo | $10/mo |
| Best for | Existing workflows | AI-first dev | Agentic flow | JetBrains users |

**Recommendation**: Start with Cursor as primary; retain VS Code for specialized tasks.

---

## Cost Summary

| Tool | Cost | Notes |
|------|------|-------|
| Perplexity Pro | $20/mo | Optional; free tier usable |
| NotebookLM | Free | Google account required |
| Claude Pro | $20/mo | Optional; pay-per-use API available |
| Cursor | $20/mo | Best ROI for AI coding |
| GitHub Copilot | $10/mo | Already have via corp license |
| Ollama | Free | Self-hosted |
| n8n | Free | Self-hosted |
| Infrastructure | $0-50/mo | Homelab + optional VPS |

**Total**: $50-100/month for full stack.

---

## Next Steps

See [SETUP_GUIDE.md](SETUP_GUIDE.md) for installation instructions.
