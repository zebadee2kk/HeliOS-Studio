# HeliOS Studio

> Personal AI R&D Lab - One-person AI-powered startup studio for discovery, research, architecture, prototyping, and deployment

## Overview

HeliOS Studio is a complete AI-powered development ecosystem that enables a solo technologist to operate like a small R&D lab or startup studio. By orchestrating multiple AI tools, local models, and automation platforms, you can:

- Discover business opportunities through systematic research
- Build and maintain a persistent knowledge base
- Design system architectures with AI assistance
- Generate working software prototypes rapidly
- Deploy and operate projects with minimal manual overhead

## Architecture

```
┌─────────────┐
│  Discovery  │  Perplexity AI → NotebookLM
└──────┬──────┘
       │
┌──────▼──────┐
│  Knowledge  │  NotebookLM + GitHub Knowledge Repos + Obsidian
└──────┬──────┘
       │
┌──────▼──────┐
│  Reasoning  │  Claude + Claude Code + Local LLMs
└──────┬──────┘
       │
┌──────▼──────┐
│   Coding    │  Cursor / VS Code + Copilot
└──────┬──────┘
       │
┌──────▼──────┐
│ Automation  │  n8n + MCP Tools + GitHub Actions
└──────┬──────┘
       │
┌──────▼──────┐
│    Infra    │  Docker + Proxmox + GitHub + Cloud
└─────────────┘
```

**New:** Tools now integrate directly via MCP servers and APIs—Claude can build n8n workflows, query NotebookLM notebooks, and manage GitHub Projects conversationally. See [Integrations Guide](docs/INTEGRATIONS.md).

## Quick Start

1. **[Architecture Overview](docs/ARCHITECTURE.md)** - Understand the system design
2. **[Tool Stack](docs/TOOL_STACK.md)** - Recommended tools for each layer
3. **[Integrations](docs/INTEGRATIONS.md)** - Direct tool connections via MCP and APIs ✨ NEW
4. **[Setup Guide](docs/SETUP_GUIDE.md)** - Step-by-step installation
5. **[Workflows](docs/WORKFLOWS.md)** - End-to-end examples
6. **[Security](docs/SECURITY.md)** - Identity separation and data protection

## Core Principles

- **AI-first**: Use AI for all code generation, architecture, and research
- **GitHub-centric**: Single source of truth for all work
- **Local + Cloud**: Balance cost, privacy, and capability
- **Automated**: Minimize manual handoffs between tools via MCP and webhooks
- **Secure**: Strong compartmentalization and data policies

## Documentation

### Getting Started
- [Architecture Overview](docs/ARCHITECTURE.md)
- [Tool Stack Recommendations](docs/TOOL_STACK.md)
- [Tool Integrations & MCP Servers](docs/INTEGRATIONS.md) ✨ NEW
- [Complete Setup Guide](docs/SETUP_GUIDE.md)
- [Hardware Recommendations](docs/HARDWARE.md)

### Operations
- [Example Workflows](docs/WORKFLOWS.md)
- [Project Management with GitHub Projects](docs/PROJECT_MANAGEMENT.md) ✨ NEW
- [Automation Patterns](docs/AUTOMATION.md)
- [Security & Identity Management](docs/SECURITY.md)

### Reference
- [MCP Integration Guide](docs/MCP.md)
- [Local LLM Strategy](docs/LOCAL_LLMS.md)
- [IDE Comparison](docs/IDE_COMPARISON.md)
- [Future Roadmap](docs/ROADMAP.md)

## Key Integrations

### MCP Servers (March 2026)

Claude Desktop can now directly:
- **Build n8n workflows** conversationally (n8n MCP server)
- **Query NotebookLM notebooks** for research (unofficial MCP server)
- **Read/write Obsidian vault** for personal knowledge (Obsidian MCP server)
- **Manage GitHub** repos, issues, PRs (official GitHub MCP server)

### Automation

n8n can now:
- **Call Perplexity API** for scheduled research queries
- **Export NotebookLM notebooks** via Apify actor
- **Receive Cursor webhooks** when agents complete
- **Update GitHub Projects** via GraphQL API

**Result:** Entire workflows from research → architecture → prototype → experiment can run with minimal manual intervention.

See [Integrations Guide](docs/INTEGRATIONS.md) for setup instructions.

## Project Management

HeliOS-Studio uses **GitHub Projects v2** as the central control tower:

- Native GitHub integration (zero sync)
- Claude can manage via GitHub MCP
- n8n automation for status updates
- Custom fields: AI Agent, Phase, Health, Cost Estimate
- Multiple views: Opportunities Board, Active Sprints, Timeline, Agent Dashboard

See [Project Management Guide](docs/PROJECT_MANAGEMENT.md) for setup and automation workflows.

## Project Ideas

This stack enables rapid development of:

- **Cybersecurity tools**: MCP security scanners, prompt-injection gateways, threat intelligence aggregators
- **Developer tooling**: Studio-in-a-box templates, local model evaluators, code quality agents
- **Automation systems**: Homelab orchestrators, experiment runners, opportunity miners
- **AI agents**: Multi-model routers, research assistants, workflow coordinators

## Ecosystem Monitoring

⚠️ **Important:** The AI tooling landscape evolves rapidly. As of March 2026, there are 8,590+ MCP servers and new integrations launch weekly.

**Monthly review checklist:**
- [ ] Check [PulseMCP.com/servers](https://www.pulsemcp.com/servers) for new MCP servers
- [ ] Review n8n integrations page for new native nodes
- [ ] Check Cursor, Claude, NotebookLM release notes
- [ ] Test existing MCP servers still work
- [ ] Update documentation with new capabilities

**Automated:** An n8n workflow creates a GitHub issue with this checklist on the first Monday of each month.

See [Integrations Guide - Maintenance Strategy](docs/INTEGRATIONS.md#maintenance-strategy) for details.

## Status

- [x] Architecture design
- [x] Tool stack research
- [x] Security framework
- [x] Direct integrations research ✨ NEW
- [x] Project management strategy ✨ NEW
- [ ] Phase 1 setup (core tools)
- [ ] Phase 2 setup (MCP + automation)
- [ ] Phase 3 setup (knowledge pipeline)
- [ ] GitHub Project creation
- [ ] First prototype project

## Contributing

This is a personal R&D lab, but if you're building something similar:

1. Fork this repo as a template for your own studio
2. Adapt the tool stack to your preferences
3. Share your learnings via issues or PRs
4. Join the conversation about AI-augmented development

## License

MIT

## Author

Built by a solo technologist with 25+ years in IT and cybersecurity, exploring the frontier of AI-augmented software development.

**Meta:** This studio documentation was researched, designed, and written with assistance from Perplexity AI, Claude, and the ecosystem of tools it describes. 🤖
