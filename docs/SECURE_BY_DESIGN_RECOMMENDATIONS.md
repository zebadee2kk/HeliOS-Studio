# Secure-by-Design Recommendations

> **Review Date:** 2026-03-09  
> **Scope Reviewed:** `README.md`, `docs/ARCHITECTURE.md`, `docs/TOOL_STACK.md`, `docs/INTEGRATIONS.md`, `docs/MCP.md`, `docs/WORKFLOWS.md`, `docs/SECURITY.md`

This document captures practical, prioritized security improvements for HeliOS Studio based on the current architecture (MCP-enabled agents, n8n automation, GitHub-centric workflows, and hybrid local/cloud AI tooling).

---

## Executive Summary

The project already has strong security foundations: identity separation, network segmentation, least-privilege guidance, and data classification are clearly documented. The highest residual risks are primarily from **integration edges**:

1. **Token handling in local configs and automation glue** (MCP server env vars, webhook/API credentials).
2. **Trust of third-party/unofficial MCP servers** (especially NotebookLM ecosystem and fast-moving MCP registries).
3. **Insufficient policy-as-code enforcement** (many controls are documented but not yet continuously verified).
4. **Automation blast radius** (n8n + GitHub + MCP chains can execute broad actions quickly if abused).

Focus first on moving from guidance to enforced controls (guardrails, CI checks, scoped credentials, and runtime isolation).

---

## Priority Recommendations

## P0 (Do now)

### 1) Standardize secret handling for MCP and automation

**Why:** Multiple integration examples use direct env token injection patterns; this is operationally convenient but high risk if machine/account is compromised.

**Recommendations:**
- Move all long-lived tokens to a secrets manager (Vault/Bitwarden Secrets/etc.) and inject at runtime.
- Prefer short-lived credentials where providers support it (OIDC, app tokens, rotating session tokens).
- Add mandatory secret scanning pre-commit and in CI (e.g., `gitleaks`/`trufflehog`).
- Add a `*.example` config pattern and block committed real configs via CI policy.

**Success criteria:**
- No plaintext high-value tokens in local JSON/yaml configs.
- Secret scan runs on every PR and push.

### 2) Enforce MCP trust policy (allowlist + pinning + review)

**Why:** MCP ecosystem growth increases supply-chain risk and inconsistent quality.

**Recommendations:**
- Maintain an explicit server allowlist with owner, source URL, commit/tag pin, and risk tier.
- Pin MCP server versions (or commit SHAs) and avoid floating installs.
- Require a lightweight security review before adding any new MCP server (permissions, network egress, update cadence).
- For unofficial servers, run in isolated containers with read-only FS and no host-level credential exposure.

**Success criteria:**
- New MCP servers cannot be added without metadata + approval.
- All installed MCP servers are version-pinned.

### 3) Harden n8n and webhook ingress

**Why:** n8n is the automation control plane; webhook abuse can trigger privileged actions.

**Recommendations:**
- Require signed webhooks (HMAC) and replay protection (timestamp + nonce).
- Place n8n behind an authenticated reverse proxy with IP allowlists where feasible.
- Separate public ingress workflows from privileged internal workflows.
- Add per-workflow risk labels and approval gates for destructive actions.

**Success criteria:**
- All incoming webhooks are authenticated and replay-protected.
- Privileged workflows are not directly internet-triggerable.

---

## P1 (Next 30 days)

### 4) Convert security guidance into policy-as-code

**Recommendations:**
- Add repo guardrails:
  - CODEOWNERS for security-sensitive docs/config.
  - branch protection + required checks.
  - dependency scanning + Dependabot + pinned action SHAs.
- Add validation scripts for:
  - forbidden path scopes in filesystem MCP config.
  - forbidden token patterns in docs/config.
  - required security headers/settings in deployment templates.

### 5) Introduce an AI action authorization model

**Recommendations:**
- Define action classes: `read`, `write`, `external_call`, `destructive`.
- Require human confirmation for `external_call` to unknown endpoints and any destructive action.
- Implement dual-control for production-impacting actions (e.g., deploy, delete, permission changes).

### 6) Build a minimum security telemetry baseline

**Recommendations:**
- Log tool-call audit trails with correlation IDs across Claude/MCP/n8n/GitHub actions.
- Alert on anomalies: off-hours write bursts, token spend spikes, new MCP server executions.
- Set retention + integrity expectations for logs used in incident response.

---

## P2 (Next 60-90 days)

### 7) Strengthen runtime isolation by default

**Recommendations:**
- Run MCP services rootless and read-only where possible.
- Apply network egress policies per service (deny-by-default + explicit allowlist).
- Use separate service identities for each integration path (GitHub, n8n, NotebookLM, Obsidian).

### 8) Formalize data governance for AI workflows

**Recommendations:**
- Add machine-checkable data labels: `public`, `internal`, `restricted`, `prohibited-for-cloud`.
- Enforce routing rules (e.g., `restricted` only to local models).
- Add mandatory redaction step for logs/prompts entering third-party LLM APIs.

### 9) Add tabletop exercises and break-glass runbooks

**Recommendations:**
- Run quarterly simulation scenarios:
  - leaked API token,
  - malicious MCP update,
  - prompt injection triggering unsafe automation.
- Maintain tested break-glass procedures with timed recovery objectives.

---

## Tool & Integration Specific Hardening

| Component | Main Risk | Recommended Hardening |
|---|---|---|
| Claude Desktop + MCP | Over-broad tool permissions | Per-server least privilege, tool allowlists, high-risk action approvals |
| n8n | Workflow abuse, webhook spoofing | Signed webhooks, segmented workflow zones, scoped credentials |
| GitHub | Token over-permission, CI supply chain | GitHub App/fine-grained tokens, required reviews, pinned action SHAs |
| NotebookLM (unofficial MCP) | Unofficial API/server trust | Sandbox execution, strict scopes, faster rotation, separate identity |
| Obsidian MCP | Local file exposure | Dedicated vault path, read-only mode where possible, path allowlists |
| Self-hosted runners | Lateral movement | Ephemeral runners, isolated network, one-job tokens |

---

## Suggested Security Backlog (Implementation Ready)

1. Add `SECURITY_BASELINE.md` with required controls and verification cadence.
2. Add CI job: secret scanning + dependency vulnerability scanning + policy checks.
3. Add `mcp-allowlist.json` (owner, version pin, scopes, review date, risk tier).
4. Add webhook verification middleware/template for n8n-trigger endpoints.
5. Add `docs/INCIDENT_RUNBOOKS.md` with step-by-step key compromise and prompt-injection response.
6. Add quarterly review template issue for access recertification and stale credential cleanup.

---

## Definition of Done (Secure-by-Design)

A workflow/integration is considered secure-by-design when all of the following are true:

- Access is least-privilege and time-bounded.
- Secrets are never hardcoded and are rotation-ready.
- High-risk actions require explicit authorization.
- Every critical action is auditable end-to-end.
- Third-party components are pinned, reviewed, and continuously monitored.
- Failure modes are tested via drills, not only documented.

