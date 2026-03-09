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
│  Knowledge  │  NotebookLM + GitHub Knowledge Repos
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

## Quick Start

1. **[Architecture Overview](docs/ARCHITECTURE.md)** - Understand the system design
2. **[Tool Stack](docs/TOOL_STACK.md)** - Recommended tools for each layer
3. **[Setup Guide](docs/SETUP_GUIDE.md)** - Step-by-step installation
4. **[Workflows](docs/WORKFLOWS.md)** - End-to-end examples
5. **[Security](docs/SECURITY.md)** - Identity separation and data protection

## Core Principles

- **AI-first**: Use AI for all code generation, architecture, and research
- **GitHub-centric**: Single source of truth for all work
- **Local + Cloud**: Balance cost, privacy, and capability
- **Automated**: Minimize manual handoffs between tools
- **Secure**: Strong compartmentalization and data policies

## Documentation

### Getting Started
- [Architecture Overview](docs/ARCHITECTURE.md)
- [Tool Stack Recommendations](docs/TOOL_STACK.md)
- [Complete Setup Guide](docs/SETUP_GUIDE.md)
- [Hardware Recommendations](docs/HARDWARE.md)

### Operations
- [Example Workflows](docs/WORKFLOWS.md)
- [Automation Patterns](docs/AUTOMATION.md)
- [Security & Identity Management](docs/SECURITY.md)

### Reference
- [MCP Integration Guide](docs/MCP.md)
- [Local LLM Strategy](docs/LOCAL_LLMS.md)
- [IDE Comparison](docs/IDE_COMPARISON.md)
- [Future Roadmap](docs/ROADMAP.md)

## Project Ideas

This stack enables rapid development of:

- **Cybersecurity tools**: MCP security scanners, prompt-injection gateways, threat intelligence aggregators
- **Developer tooling**: Studio-in-a-box templates, local model evaluators, code quality agents
- **Automation systems**: Homelab orchestrators, experiment runners, opportunity miners
- **AI agents**: Multi-model routers, research assistants, workflow coordinators

## Status

- [x] Architecture design
- [x] Tool stack research
- [x] Security framework
- [ ] Phase 1 setup (core tools)
- [ ] Phase 2 setup (MCP + automation)
- [ ] Phase 3 setup (knowledge pipeline)
- [ ] First prototype project

## License

MIT

## Author

Built by a solo technologist with 25+ years in IT and cybersecurity, exploring the frontier of AI-augmented software development.
