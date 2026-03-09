# Project Management for HeliOS-Studio

> **Last Updated:** March 9, 2026

This guide covers the project management strategy for HeliOS-Studio, optimized for solo operation with heavy AI assistance.

## Strategy: GitHub Projects v2

For a solo technologist operating an AI R&D lab, **GitHub Projects v2** is the optimal choice over Linear, Jira, or other PM tools.

### Why GitHub Projects

✅ **Native GitHub integration**  
Issues, PRs, and commits auto-link with zero sync overhead. No impedance mismatch between your code and your work tracking.

✅ **MCP and API support**  
Claude can create and update project items directly via GitHub MCP server. Your AI assistant becomes your project manager.

✅ **Automation built-in**  
GitHub Actions can update project fields (status, priority) on PR merge, label changes, or any GitHub event.

✅ **Free and unlimited**  
No per-seat costs for personal repos. Scale infinitely without budget concerns.

✅ **Multiple views from one dataset**  
Kanban board, table, roadmap timeline, and custom views all from the same underlying issues.

✅ **Lightweight**  
No external tool sync, no additional login, no separate database to maintain.

### When NOT to use GitHub Projects

❌ **Complex enterprise workflows** with approval chains, SLAs, and compliance tracking (use Jira)  
❌ **Cross-company collaboration** where external partners can't access your GitHub (use Linear or Asana)  
❌ **Time tracking is critical** and you need built-in timesheet features (use Toggl + GitHub Projects)  
❌ **You prefer Linear's UI/UX** and are willing to pay $8/user/mo and maintain sync  

---

## Setup Guide

### 1. Create Project

1. Go to [github.com/users/zebadee2kk/projects](https://github.com/users/zebadee2kk/projects)
2. Click **New project**
3. Choose **Table** view to start
4. Name: `HeliOS-Studio Control Tower`
5. Description: `Central command for AI R&D operations`

### 2. Link Repositories

Add all HeliOS-related repos:
- `HeliOS-Studio` (this repo)
- `hamnet` (infrastructure)
- `control-tower`
- `ai-cost-tracker`
- `sentinelforge`
- Any `kb-*` knowledge repos
- Any prototype repos

**How:** In project settings → Add repositories → Select from dropdown

### 3. Configure Custom Fields

Add these fields beyond the default `Status`, `Assignee`, `Labels`:

| Field Name | Type | Options | Purpose |
|------------|------|---------|----------|
| `AI Agent` | Select | Claude, Copilot, Cursor, n8n, Manual | Track which AI did the work |
| `Research Source` | Text | — | Link to NotebookLM notebook or research doc |
| `Cost Estimate` | Number | — | Estimated $ for API calls and compute |
| `Phase` | Select | Discovery, Research, Architecture, Prototype, Deploy, Maintain | Lifecycle stage |
| `Health` | Select | Healthy, At Risk, Dead | Project health based on experiments |
| `Priority` | Select | P0 (Critical), P1 (High), P2 (Medium), P3 (Low) | Work prioritization |
| `Effort` | Select | XS (1-2h), S (1d), M (1w), L (2w), XL (1mo+) | Time estimate |

**How:** Project settings → Fields → New field

### 4. Create Views

#### View 1: Opportunities Board (Kanban)

**Purpose:** Triage and validate new opportunities from research

**Setup:**
- Layout: Board
- Group by: `Status`
- Columns: `Backlog` → `Validated` → `Planned` → `Archived`
- Filter: `Phase: Discovery OR Research`
- Sort: By `Priority` (P0 first)

**Workflow:**
1. New research exports create items in `Backlog`
2. You review and move to `Validated` if worth pursuing
3. Claude generates architecture and moves to `Planned`
4. When work starts, item moves to Active Sprints view

#### View 2: Active Sprints (Table)

**Purpose:** Track current work across all projects

**Setup:**
- Layout: Table
- Filter: `Status: In Progress OR Status: In Review`
- Group by: `AI Agent`
- Sort: By `Priority`, then `Updated` (most recent first)
- Visible columns: Title, Status, AI Agent, Phase, Health, Cost Estimate, Assignee

**Workflow:**
- Everything actively being built shows here
- Grouped by which AI is doing the work
- Daily review: check Health and update Status

#### View 3: Research Timeline (Roadmap)

**Purpose:** Visualize project pipeline from idea to deployment

**Setup:**
- Layout: Roadmap
- Group by: `Phase`
- Date field: Use `Start date` and `Target date` (add these fields if needed)
- Filter: `Status: NOT Archived`

**Workflow:**
- Visual timeline of all active initiatives
- Drag items to adjust schedule
- Identify bottlenecks (too many in one phase)

#### View 4: AI Agent Dashboard (Table)

**Purpose:** Monitor what each AI is working on and productivity

**Setup:**
- Layout: Table
- Group by: `AI Agent`
- Filter: `Status: In Progress`
- Sort: By `Updated` (most recent first)
- Visible columns: Title, Status, Phase, Effort, Cost Estimate

**Workflow:**
- See at a glance: Claude has 3 items, Cursor has 2, n8n has 5 workflows
- Identify underutilized AI agents
- Track cost per agent

---

## Automation Workflows

### Workflow 1: New Research → Create Project Item

**Trigger:** n8n detects new file in `kb-opportunities-*` repo

```
[GitHub Webhook: Push to kb-opportunities-* repo]
  ↓
[Filter: Only new markdown files]
  ↓
[Parse markdown: Extract title and summary]
  ↓
[GitHub API: Create project item]
  │
  └─> Title: [Extracted title]
      Body: [Summary + link to full research]
      Status: Backlog
      Phase: Discovery
      Health: Healthy
      Research Source: [Link to NotebookLM or repo]
  ↓
[Slack: Post to #opportunities channel]
```

**Implementation:**
```javascript
// n8n HTTP Request node to GitHub GraphQL API
const mutation = `
mutation {
  addProjectV2ItemById(
    input: {
      projectId: "PVT_kwDOAB..."
      contentId: "I_kwDOAB..."  # Issue node ID
    }
  ) {
    item { id }
  }
}`;

// Then update custom fields
const updateFields = `
mutation {
  updateProjectV2ItemFieldValue(
    input: {
      projectId: "PVT_kwDOAB..."
      itemId: "PVTI_lADOAB..."
      fieldId: "PVTF_lADOAB..."  # Phase field ID
      value: { singleSelectOptionId: "discovery_option_id" }
    }
  ) {
    projectV2Item { id }
  }
}`;
```

---

### Workflow 2: Cursor Agent Completes → Update Status

**Trigger:** Cursor webhook when agent finishes task

```
[Cursor Webhook: Agent completed]
  ↓
[Parse payload: Get repo and changed files]
  ↓
[GitHub API: Find project items for this repo]
  ↓
[Filter: Items with Status="In Progress"]
  ↓
[GitHub API: Update Status="Ready for Review"]
  ↓
[GitHub API: Add comment: "Cursor agent completed. Files changed: ..."]
  ↓
[Slack: Notify #dev-updates]
```

---

### Workflow 3: PR Merged → Update Phase

**Trigger:** GitHub webhook when PR merged to main

```
[GitHub Webhook: Pull request merged]
  ↓
[Extract PR labels]
  ↓
[If label contains "prototype"]:
    Update Phase: Deploy
[If label contains "architecture"]:
    Update Phase: Prototype
  ↓
[GitHub API: Update project item Phase field]
  ↓
[If Phase changed to "Deploy"]:
    Trigger deployment workflow
```

---

### Workflow 4: Weekly Lab Report → Update Health

**Trigger:** Schedule (Friday 5pm)

```
[Schedule: Friday 5pm]
  ↓
[GitHub API: Get all items with Phase="Prototype" or "Deploy"]
  ↓
[For each item]:
  [NotebookLM MCP: Query experiment logs for this project]
    ↓
  [Claude API: "Analyze experiment results and recommend: keep/kill/pivot"]
    ↓
  [Parse response]
    ↓
  [GitHub API: Update Health field]
    │
    ├─> "keep" → Healthy
    ├─> "pivot" → At Risk
    └─> "kill" → Dead
    ↓
  [GitHub API: Add comment with Claude's analysis]
  ↓
[Generate weekly summary report]
  ↓
[GitHub: Create issue "Weekly Lab Report - [Date]"]
  ↓
[Slack: Post summary]
```

---

### Workflow 5: Cost Tracking → Update Budget Field

**Trigger:** Schedule (daily midnight)

```
[Schedule: Daily 00:00]
  ↓
[Query ai-cost-tracker database]
  ↓
[Group costs by project/repo]
  ↓
[For each project]:
  [GitHub API: Find project item]
    ↓
  [Update Cost Estimate field with actual spend]
    ↓
  [If cost > estimate * 1.5]:
    Update Health: At Risk
    Add comment: "Cost overrun: $X over estimate"
  ↓
[Generate cost report]
  ↓
[GitHub: Create issue "Daily Cost Report"]
```

---

## Claude-Driven Project Management

### Using GitHub MCP for PM Tasks

With the GitHub MCP server installed, Claude can manage your projects conversationally:

**Example prompts:**

```
You: "Claude, create a new project item for the MCP security scanner idea. 
Set Phase to Architecture, Priority to P1, and assign to me. 
Add a link to the kb-opportunities-mcp-security repo as the research source."

Claude: [Uses GitHub MCP to create item with all fields set]
```

```
You: "Show me all items with Health=At Risk"

Claude: [Queries project, returns list with links and summaries]
```

```
You: "The sentinelforge prototype is working well. Move it from Prototype to Deploy phase 
and update health to Healthy. Add a comment summarizing this week's experiments."

Claude: [Updates fields and adds detailed comment via GitHub MCP]
```

### Weekly Review with Claude

**Friday afternoon routine:**

1. You: "Claude, generate a weekly lab report. Query the NotebookLM experiment logs, 
   analyze all projects in Prototype or Deploy phase, and recommend keep/kill/pivot 
   for each. Update project Health fields and create a summary issue."

2. Claude:
   - Uses NotebookLM MCP to read experiment notebooks
   - Analyzes results (success metrics, cost, velocity)
   - Uses GitHub MCP to update Health fields
   - Creates formatted issue with recommendations
   - You review and approve kill/pivot decisions

3. You action the recommendations in 15 minutes instead of 2 hours of manual review.

---

## Maintenance

### Daily
- ✅ Check Active Sprints view (2 min)
- ✅ Review new items in Opportunities Board (5 min)
- ✅ Update status of items you worked on manually (if AI didn't auto-update)

### Weekly
- ✅ Run weekly lab report (automated, just review)
- ✅ Review Research Timeline for bottlenecks
- ✅ Archive completed/dead projects
- ✅ Check AI Agent Dashboard for underutilization

### Monthly
- ✅ Review custom field usage (are we tracking the right things?)
- ✅ Audit automation workflows (are they still working?)
- ✅ Update this doc with new learnings

---

## GitHub Projects API Reference

### Get Project ID

```bash
gh api graphql -f query='
{
  user(login: "zebadee2kk") {
    projectsV2(first: 10) {
      nodes {
        id
        title
      }
    }
  }
}'
```

### Get Field IDs

```bash
gh api graphql -f query='
{
  node(id: "PVT_kwDOAB...") {
    ... on ProjectV2 {
      fields(first: 20) {
        nodes {
          ... on ProjectV2SingleSelectField {
            id
            name
            options {
              id
              name
            }
          }
        }
      }
    }
  }
}'
```

### Create Project Item

```bash
gh api graphql -f query='
mutation {
  addProjectV2ItemById(input: {
    projectId: "PVT_kwDOAB..."
    contentId: "I_kwDOAB..."  # Issue/PR node ID
  }) {
    item { id }
  }
}'
```

### Update Custom Field

```bash
gh api graphql -f query='
mutation {
  updateProjectV2ItemFieldValue(input: {
    projectId: "PVT_kwDOAB..."
    itemId: "PVTI_lADOAB..."
    fieldId: "PVTF_lADOAB..."  # Custom field ID
    value: {
      singleSelectOptionId: "option_id"  # For select fields
      # OR text: "some text"  # For text fields
      # OR number: 42  # For number fields
    }
  }) {
    projectV2Item { id }
  }
}'
```

---

## Alternative: Linear

If you decide GitHub Projects is too basic and want Linear:

### Linear Advantages
- ⭐ Best-in-class UI/UX (keyboard shortcuts, instant search)
- ⭐ Built-in cycles/sprints with velocity tracking
- ⭐ Triage inbox with smart prioritization
- ⭐ Excellent GitHub sync (2-way, real-time)

### Linear Disadvantages
- ❌ $8/user/mo cost
- ❌ Requires sync tool (built-in, but still a dependency)
- ❌ No direct MCP support (would need custom bridge)
- ❌ Another login, another tool to maintain

### Setup if choosing Linear

1. Sign up at linear.app
2. Enable GitHub integration in Linear settings
3. Configure sync rules (which repos, label mapping)
4. Build n8n workflows to:
   - Create Linear issues from research exports
   - Update Linear status from Cursor webhooks
   - Query Linear API for weekly reports

**My recommendation:** Start with GitHub Projects. Upgrade to Linear only if you find yourself fighting GitHub's limitations after 3 months of use.

---

## Resources

- [GitHub Projects documentation](https://docs.github.com/en/issues/planning-and-tracking-with-projects)
- [GitHub Projects API (GraphQL)](https://docs.github.com/en/issues/planning-and-tracking-with-projects/automating-your-project/using-the-api-to-manage-projects)
- [GitHub Agentic Workflows blog](https://github.blog/ai-and-ml/automate-repository-tasks-with-github-agentic-workflows/)
- [Automate GitHub with AI agents guide](https://fleeceai.app/blog/automate-github-with-ai-agents-2026)
- [Linear vs GitHub Projects comparison](https://everhour.com/blog/linear-vs-jira/) (Jira article but covers Projects too)

---

## Next Steps

1. ✅ Document project management strategy (this file)
2. ⬜ Create GitHub Project "HeliOS-Studio Control Tower"
3. ⬜ Add custom fields (AI Agent, Phase, Health, etc.)
4. ⬜ Create 4 views (Opportunities, Active, Timeline, Agent Dashboard)
5. ⬜ Build n8n automation workflows
6. ⬜ Test Claude → GitHub Projects via MCP
7. ⬜ Run first weekly lab report
8. ⬜ Evaluate after 1 month: keep GitHub Projects or upgrade to Linear
