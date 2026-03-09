# HeliOS Studio Secure-by-Design Blueprint

> **Objective:** Design the full HeliOS ecosystem so security is an architectural property (not an afterthought) across MCP, n8n, GitHub, local models, and cloud APIs.

This blueprint is tied to the planned stack and interactions described in `ARCHITECTURE.md`, `INTEGRATIONS.md`, `MCP.md`, `WORKFLOWS.md`, and `SECURITY.md`.

---

## 1) Security Architecture for the Planned System

### 1.1 Trust Zones (enforce explicitly)

Define and enforce these zones in network, identity, and runtime policy:

1. **Zone A – Studio Control Plane**
   - Claude Desktop / Claude Code
   - Cursor / VS Code
   - Local workstation + studio browser profile
   - Highest operator privileges

2. **Zone B – Automation Plane**
   - n8n + webhook ingress
   - Orchestration logic and scheduled jobs
   - Medium-high privilege, highest automation blast radius

3. **Zone C – Tooling/Data Plane**
   - MCP servers (filesystem, GitHub, n8n, Obsidian, NotebookLM)
   - Ollama runtime and model endpoints
   - Direct data read/write paths

4. **Zone D – External Service Plane**
   - GitHub Cloud APIs
   - Perplexity, Anthropic, NotebookLM/Apify, Slack, other SaaS
   - Untrusted by default, egress controlled

5. **Zone E – Sensitive/Restricted Data Plane**
   - Secrets managers, backup snapshots, incident artifacts
   - Must be inaccessible to general-purpose agents by default

### Required baseline
- **Default deny** between zones (allowlist only required paths).
- **Different identities per zone/service** (no shared admin tokens).
- **Cryptographic webhook/API authentication** at every cross-zone hop.

---

### 1.2 Critical Trust Boundaries in HeliOS Flows

These boundaries must be treated as attack surfaces:

- `GitHub Webhook -> n8n` (payload authenticity + replay prevention)
- `n8n -> Claude API / Perplexity / Apify` (data exfil and prompt injection)
- `Claude Desktop/Code -> MCP Servers` (tool overreach and privilege abuse)
- `MCP filesystem -> local repo/vault` (path traversal and unintended writes)
- `n8n/MCP -> GitHub write actions` (supply-chain tampering)
- `NotebookLM unofficial MCP / Apify` (third-party trust + token theft)

---

## 2) Threat Model Expansion (AI-Forward, 2026+)

In addition to classic threats, explicitly model AI-native threats:

1. **Prompt injection across tool chains**
   - Malicious issue/comment/page causes agent to perform privileged operations.

2. **Context poisoning / retrieval poisoning**
   - Untrusted docs in NotebookLM/Obsidian/GitHub alter agent decision quality.

3. **Agent tool escalation**
   - Benign workflow triggers hidden write/delete/external-call capability.

4. **Model supply-chain and jailbreak drift**
   - New model behavior bypasses existing safeguards after version updates.

5. **Autonomous loop abuse**
   - Scheduled/recursive workflows create high-speed destructive propagation.

6. **Indirect exfiltration**
   - Secrets encoded into logs, summaries, commit messages, or downstream payloads.

---

## 3) Secure-by-Design Control Pattern (Mandatory)

Every integration and workflow should implement this control pattern:

`Authenticate -> Authorize -> Validate -> Execute in Sandbox -> Observe -> Recover`

### 3.1 Authenticate
- Signed webhooks (HMAC-SHA256), timestamp + nonce replay window.
- mTLS or equivalent service auth for internal service-to-service traffic where feasible.
- OIDC/workload identity over static tokens whenever possible.

### 3.2 Authorize
- Policy engine for action classes: `read`, `write`, `external_call`, `destructive`.
- Human approval gates for:
  - destructive operations,
  - production deploys,
  - new external endpoints,
  - permission or credential changes.

### 3.3 Validate
- Treat all upstream content as untrusted (issues, PR comments, web content, notebook text).
- Strict JSON schema validation for webhook and tool payloads.
- Allowlist command templates and endpoint destinations.

### 3.4 Execute in Sandbox
- MCP servers in containers, rootless, read-only FS, seccomp/apparmor.
- Per-server network egress allowlists.
- Filesystem MCP restricted to explicit directories only.

### 3.5 Observe
- End-to-end trace ID across GitHub -> n8n -> AI -> MCP -> GitHub.
- Log decision prompts, tool invocations, target resources, and approvals.
- Alert on anomalies (off-hours writes, token spikes, new server binaries).

### 3.6 Recover
- One-click key revocation + rotation playbooks.
- Automated containment (disable workflow/token/server on anomaly severity threshold).
- Snapshot/restore tested for n8n, MCP infra, and repos.

---

## 4) Tool-by-Tool Hardening Requirements

### 4.1 MCP Ecosystem

### Minimum controls
- Maintain `mcp-allowlist` inventory with owner, repo URL, version pin/SHA, permissions, last review date, risk tier.
- No unreviewed MCP server installation in production environment.
- Block floating installs (`latest`, unpinned `npx -y` in production contexts).

### Unofficial MCP (NotebookLM especially)
- Isolate to dedicated container/VM and dedicated low-privilege account.
- Separate tokens from core studio GitHub/n8n credentials.
- Faster rotation cadence (monthly) and stricter monitoring.

### 4.2 n8n (Automation Hub)

### Minimum controls
- Separate **public ingress workflows** from **privileged internal workflows**.
- All webhook triggers must pass signature + replay validation node before business logic.
- Store credentials in encrypted secret store only; never in workflow JSON exports.
- Add kill-switch workflow to disable high-risk automations quickly.

### High-risk workflow controls
- Require manual approval node for destructive writes/deploys.
- Concurrency + rate limits to prevent automation storms.
- Destination allowlists for HTTP Request nodes.

### 4.3 GitHub (Source of Truth + CI/CD)

### Minimum controls
- Prefer GitHub Apps/fine-grained tokens over classic PATs.
- Enforce branch protection, required reviews, status checks, signed commits/tags where possible.
- Pin GitHub Actions by commit SHA; enable Dependabot and code/dependency scanning.
- For self-hosted runners, use ephemeral runners and one-job credentials.

### 4.4 Claude/Cursor Agent Paths

### Minimum controls
- Explicit tool allowlist by agent persona/workflow.
- Prompt boundary hardening: separate system policy from untrusted user/retrieved content.
- Block direct execution of generated shell commands without policy validation.
- Require confirmation for operations crossing trust zones.

### 4.5 Ollama / Local Model Runtime

### Minimum controls
- Keep local model endpoint private to studio VLAN; no public exposure.
- Pin model versions used in automation-critical tasks.
- Add regression safety tests when changing model/version for high-impact workflows.

---

## 5) Data Security-by-Design

### 5.1 Data classification tags (machine enforceable)

Use mandatory tags for each artifact/payload:
- `public`
- `studio-internal`
- `restricted`
- `prohibited-for-cloud`

### 5.2 Routing policy
- `restricted` and `prohibited-for-cloud` data may only be processed by approved local models/services.
- Third-party LLM/API calls must include redaction stage + DLP checks.
- Logs must be secret-scrubbed before storage/export.

### 5.3 Provenance + integrity
- Track source provenance for NotebookLM/Obsidian/GitHub imported context.
- Mark low-trust sources to reduce agent action authority.

---

## 6) Future-Resilient AI Security Guardrails

To harden against evolving AI threats:

1. **Policy isolation from model behavior**
   - Keep security policy enforcement external to the model output.

2. **Canary secrets and honey workflows**
   - Detect exfil attempts and unauthorized traversal early.

3. **Adversarial test suite**
   - Maintain prompt-injection, context-poisoning, and unsafe-tool-use test corpus.

4. **Model/update gate**
   - No model/tool version upgrade without security regression tests.

5. **Continuous red-team cadence**
   - Quarterly AI red-team simulation across key workflows.

---

## 7) 90-Day Hardening Plan (Concrete)

### Days 0-14 (Foundational lock-in)
- Build zone diagram + enforce firewall/egress rules matching trust zones.
- Implement webhook signature + replay checks on all inbound webhooks.
- Create MCP allowlist inventory and pin current server versions.
- Enable secret scanning and dependency scanning in CI.

### Days 15-45 (Control enforcement)
- Add authorization policy for action classes and approval gates.
- Segment n8n workflows into public vs privileged lanes.
- Introduce end-to-end trace IDs and central log correlation.
- Implement destination allowlists for outbound HTTP calls.

### Days 46-90 (Resilience + validation)
- Run first AI security tabletop + technical simulation.
- Add adversarial prompt/context test suite to CI.
- Validate break-glass procedures for key compromise and workflow takeover.
- Define SLOs for detection, containment, and recovery.

---

## 8) Security Acceptance Criteria (No-Compromise Bar)

A workflow is approved only if all are true:

- Identity is least-privilege and scoped to a single function.
- Secrets are managed centrally and rotation is automated/tested.
- Input is authenticated, validated, and policy-checked before action.
- High-risk actions require explicit human authorization.
- Execution context is sandboxed with restricted file/network access.
- Full audit trail exists with traceable approvals and outcomes.
- Failure/abuse paths are tested (not only documented).

---

## 9) Recommended New Repo Artifacts

To operationalize this blueprint, add:

1. `docs/SECURITY_ARCHITECTURE.md` (zones, trust boundaries, data flows)
2. `docs/AI_THREAT_MODEL.md` (threat trees + abuse cases)
3. `docs/AI_SECURITY_TEST_PLAN.md` (injection and escalation tests)
4. `security/mcp-allowlist.json` (pinning + review metadata)
5. `security/policies/action-authorization.rego` (or equivalent policy spec)
6. `docs/INCIDENT_RUNBOOKS.md` (key compromise, poisoned context, workflow hijack)

