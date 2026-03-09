# HeliOS-Studio Changelog

All notable changes and enhancements to this project will be documented in this file.

## [Unreleased]

### Added

## [2026-03-09] - Major Integration & Automation Update

### Added

#### Documentation
- **[INTEGRATIONS.md](docs/INTEGRATIONS.md)** - Comprehensive guide to direct tool integrations
  - 4 critical MCP servers (n8n, NotebookLM, Obsidian, GitHub)
  - Native n8n integrations (Perplexity, Claude, NotebookLM via Apify)
  - Cursor webhooks setup
  - Complete integration matrix showing all tool connections
  - Maintenance strategy for rapid ecosystem evolution
  
- **[PROJECT_MANAGEMENT.md](docs/PROJECT_MANAGEMENT.md)** - Full project management strategy
  - GitHub Projects v2 setup guide with custom fields
  - 4 pre-configured views (Opportunities, Active Sprints, Timeline, Agent Dashboard)
  - 5 automation workflows (research → project creation, status updates, health tracking)
  - Claude-driven project management examples via GitHub MCP
  - GitHub Projects API reference with GraphQL examples
  - Linear alternative comparison for future consideration

#### Workflows
- **[monthly-ecosystem-review.json](workflows/monthly-ecosystem-review.json)** - Automated monitoring
  - n8n workflow template for monthly tool ecosystem review
  - Auto-scrapes PulseMCP, n8n integrations, Cursor releases
  - Creates GitHub issue with comprehensive review checklist
  - Optional Slack notification
  - Runs first Monday of each month at 9am GMT

#### README Updates
- Added direct integration callouts (MCP servers, n8n automation)
- Added project management section
- Added ecosystem monitoring warning and monthly review process
- Added new documentation links
- Updated status checklist with new deliverables
- Added meta-note about AI-assisted development

### Context

**Why this update matters:**

The AI tooling ecosystem is evolving rapidly. As of March 2026:
- 8,590+ MCP servers exist (updated daily)
- MCP enables Claude to directly control n8n, GitHub, NotebookLM, Obsidian
- n8n can now call Perplexity API and export NotebookLM notebooks
- Cursor supports webhooks for agent automation
- These integrations eliminate 80%+ of manual handoffs between tools

**Key capabilities unlocked:**

1. **Claude → n8n (MCP)**: Build automation workflows conversationally
2. **Claude → NotebookLM (MCP)**: Query research notebooks directly in chat
3. **Claude → GitHub Projects (MCP)**: Manage work items and status updates via AI
4. **n8n → Everything**: Orchestrate Perplexity queries, NotebookLM exports, GitHub webhooks, Cursor triggers
5. **Automated monitoring**: Monthly ecosystem reviews ensure you don't miss important updates

**Result:** End-to-end workflows (Opportunity → Research → Architecture → Prototype → Deploy) can run with minimal manual intervention.

### Breaking Changes

None. These are additive enhancements.

### Migration Guide

No migration needed. Existing setup continues to work. New features are opt-in:

1. Install MCP servers (see INTEGRATIONS.md)
2. Create GitHub Project (see PROJECT_MANAGEMENT.md)
3. Import n8n workflow (see workflows/ directory)
4. Configure automation (see both guides)

---

## [2026-03-08] - Initial Repository Setup

### Added

#### Core Documentation
- README.md with project overview
- docs/ARCHITECTURE.md - System design and layered architecture
- docs/TOOL_STACK.md - Recommended tools for each layer
- docs/HARDWARE.md - Workstation and homelab specifications
- docs/MCP.md - Model Context Protocol integration guide
- docs/SECURITY.md - Identity separation and operational security
- docs/SETUP_GUIDE.md - Phased installation steps
- docs/WORKFLOWS.md - End-to-end example workflows

#### License
- MIT License

---

## Versioning Strategy

This project follows a date-based versioning scheme (YYYY-MM-DD) given the rapid pace of AI tooling evolution.

**Major updates:** New integrations, architecture changes, breaking changes  
**Minor updates:** Documentation improvements, workflow examples, bug fixes  
**Ecosystem reviews:** Monthly on first Monday (automated)

---

## Maintenance Notes

**Review frequency:** Monthly (automated via n8n workflow)  
**Last ecosystem review:** 2026-03-09 (initial setup)  
**Next scheduled review:** 2026-04-07 (first Monday of April)  

**Tools to monitor:**
- [PulseMCP.com/servers](https://www.pulsemcp.com/servers) - MCP server directory (8,590+ servers)
- [n8n.io/integrations](https://n8n.io/integrations) - Native n8n nodes
- [releasebot.io/updates/cursor](https://releasebot.io/updates/cursor) - Cursor IDE releases
- [Anthropic changelog](https://docs.anthropic.com/changelog) - Claude API updates
- [NotebookLM updates](https://notebooklm.google.com/) - Google AI updates

---

## Contributing

This is a personal R&D lab. If you're building something similar:

1. Fork as template for your studio
2. Adapt to your tool preferences
3. Share learnings via issues/PRs
4. Star if you find it useful

---

## Contact

Built by [@zebadee2kk](https://github.com/zebadee2kk) - 25+ years IT/cybersecurity professional exploring AI-augmented development.
