# HeliOS Studio Security Framework

Security guidelines for identity separation, data protection, and AI agent isolation.

---

## Threat Model

### Assets

- **Corporate data**: Work systems, client data, Nestlé compliance obligations
- **Personal data**: Banking, email, social accounts, family info
- **Studio data**: Project code, research, API keys, experiment results
- **Credentials**: Passwords, API keys, SSH keys, tokens

### Threats

1. **Data leakage**: Personal/corporate data sent to third-party LLMs
2. **Credential compromise**: API keys stolen from config files or logs
3. **Prompt injection**: Malicious inputs causing unintended AI actions
4. **Cross-contamination**: Studio compromise affecting personal or work systems
5. **Supply chain**: Compromised MCP servers or n8n workflows

### Mitigations

See sections below.

---

## Identity Separation Strategy

### Three-Tier Model

#### 1. Corporate Identity

**Use for**: 
- Employer systems only
- M365, Teams, corporate VPN
- MDM-managed laptop

**Never use for**:
- Studio projects
- Side projects
- Personal AI tools

**Devices**: Work laptop only, separate browser profile

#### 2. Personal Identity

**Use for**:
- Personal email, banking, social media
- Family and friend communications
- Personal shopping and services

**Never use for**:
- Studio projects (except billing/payment)
- Work systems

**Devices**: Personal laptop/phone, separate browser profile

#### 3. Studio Identity (NEW)

**Create**:
- Email: `you@yourstudio.com` (custom domain)
- GitHub org: `your-studio`
- Accounts: Perplexity, Claude, NotebookLM, n8n, etc.

**Use for**:
- All HeliOS Studio work
- AI R&D projects
- Open-source contributions under studio brand

**Devices**: Studio VM or dedicated workstation, separate browser profile

**Benefits**:
- Clear legal/commercial separation
- Blast radius containment
- Professional branding
- Honest attribution (not anonymous)

---

## Device and Network Separation

### Recommended Setup

```
┌─────────────────┐
│  Work Laptop    │  Corporate domain, MDM, VPN to work
│  (Corporate ID) │  NEVER used for studio
└─────────────────┘

┌─────────────────┐
│ Personal Laptop │  Personal email, banking, social
│  (Personal ID)  │  NEVER used for studio (except billing)
└─────────────────┘

┌─────────────────┐
│  Studio VM      │  Proxmox VM, studio identity only
│  (Studio ID)    │  Cursor, Claude Desktop, NotebookLM browser
└────────┬────────┘
         │
         ↓ (studio VLAN)
┌────────────────────────────────┐
│      Homelab Services          │
│  • Ollama (AI Core VM)         │
│  • n8n (Automation Hub VM)     │
│  • GitHub Runners              │
│  • Monitoring                  │
└────────────────────────────────┘
```

### Network Segmentation

**Studio VLAN** (e.g. `10.100.0.0/24`):
- Studio VM
- AI Core VM (Ollama)
- Automation Hub VM (n8n)
- GitHub self-hosted runners

**Firewall rules**:
- Studio VLAN → Internet: Allow (outbound only)
- Studio VLAN → Corporate VLAN: **DENY**
- Studio VLAN → Personal devices: **DENY**
- Management subnet → Studio VLAN: Allow (for admin)

### Browser Profiles

**Create separate profiles** in Chrome/Firefox/Edge:

1. **Corp Profile**: Work email, M365, Teams
2. **Personal Profile**: Personal email, banking, social
3. **Studio Profile**: Studio email, GitHub studio org, AI tools

**Never cross-login** between profiles.

**Use**: Each profile has its own cookies, cache, extensions, and bookmarks.

---

## Credential Management

### Password Manager

**Setup**:
- Use 1Password, Bitwarden, or similar with separate **vaults**:
  - `Corporate` vault
  - `Personal` vault
  - `Studio` vault

**Rules**:
- Unique password for every account
- 20+ character random passwords
- MFA enabled everywhere possible

### API Keys and Secrets

**Never**:
- Commit to Git (even private repos)
- Store in plaintext config files
- Log to stdout/stderr
- Pass via URL parameters

**Best practices**:

#### 1. Environment Variables

```bash
# .env (gitignored)
CLAUDE_API_KEY=sk-ant-...
GITHUB_TOKEN=ghp_...
```

```python
import os
from dotenv import load_dotenv

load_dotenv()
api_key = os.getenv("CLAUDE_API_KEY")
```

#### 2. Docker Secrets

```yaml
version: '3.8'
services:
  app:
    image: myapp
    secrets:
      - claude_key

secrets:
  claude_key:
    file: ./secrets/claude_key.txt
```

```python
with open('/run/secrets/claude_key') as f:
    api_key = f.read().strip()
```

#### 3. HashiCorp Vault (Advanced)

```bash
# Store
vault kv put secret/studio/claude api_key=sk-ant-...

# Retrieve
vault kv get -field=api_key secret/studio/claude
```

#### 4. GitHub Secrets (for Actions)

```yaml
# .github/workflows/deploy.yml
steps:
  - name: Deploy
    env:
      CLAUDE_KEY: ${{ secrets.CLAUDE_API_KEY }}
    run: ./deploy.sh
```

### Key Rotation

- Rotate all API keys **quarterly**
- Immediately rotate if:
  - Key appears in logs
  - Team member leaves
  - Suspicious activity detected

---

## Data Classification

### Allowed in Studio (Safe for Third-Party LLMs)

✅ Public documentation  
✅ Open-source code  
✅ Your own non-sensitive notes  
✅ Anonymized/synthetic test data  
✅ General technical questions  

### Never Allowed

❌ Corporate data (emails, docs, code, configs)  
❌ Client data  
❌ PII (names, emails, addresses, phone numbers)  
❌ PHI/PCI (health, payment card data)  
❌ Secrets (passwords, API keys, tokens)  
❌ Private network topology or configs  

### Gray Area (Use Local LLMs Only)

⚠️ Logs (may contain sensitive patterns)  
⚠️ Metrics (may reveal private system details)  
⚠️ Draft designs (not yet public)  

**Rule**: If you wouldn't post it on a public blog, use local LLMs or redact before cloud LLMs.

---

## MCP Security

### Principle: Least Privilege

Each MCP server should have **minimal** access to resources.

### Filesystem MCP

**Sandbox**: Only `/workspace/helios-studio` directory

```json
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": [
        "-y",
        "@modelcontextprotocol/server-filesystem",
        "/home/user/workspace/helios-studio"
      ]
    }
  }
}
```

**Never**: Give access to `/`, `~`, or sensitive directories like `~/.ssh`, `~/.aws`

### GitHub MCP

**Fine-grained token**: Scoped to studio org only

**Permissions**:
- Contents: Read and write (specific repos only)
- Issues: Read and write
- Pull requests: Read and write
- Metadata: Read-only

**Never**: Use personal access token (classic) with full repo access

### Custom MCP Servers

**Run in containers**:

```yaml
services:
  mcp-n8n:
    build: ./mcp-servers/n8n
    read_only: true
    networks:
      - mcp_net
    environment:
      - N8N_URL=http://n8n:5678
    secrets:
      - n8n_api_key
```

**Isolate**: No direct internet access; communicate via internal network only

---

## AI Agent Security

### Prompt Injection Mitigations

**Input validation**:
- Allowlist expected commands/patterns
- Reject inputs with suspicious tokens (e.g., "ignore previous instructions")
- Sanitize file paths and shell commands

**Output filtering**:
- Block responses containing secrets (regex for API keys, tokens)
- Redact PII before logging

**Tool restrictions**:
- Define explicit tool allowlists per agent
- Require human approval for high-risk actions:
  - Delete operations
  - Public deployments
  - External API calls

**Example (n8n workflow)**:

```
[User Input] → [Validation Node] → [Claude API] → [Output Filter] → [Action]
                    ↓ reject                        ↓ sanitize
               [Log & Alert]                     [Log & Alert]
```

### Audit Logging

**Log all AI actions**:
- Prompts sent (redact secrets)
- Tool calls made
- Files accessed/modified
- API endpoints hit
- User who triggered

**Centralize logs**:
- Ship to ELK/Loki/Grafana Loki
- Retention: 90 days minimum
- Alerting on anomalies:
  - Unusual file access patterns
  - Off-hours API calls
  - High token usage
  - Failed authentication attempts

### Rate Limiting

**Per-agent limits**:
- Max API calls per hour: 100
- Max tokens per day: 500k
- Max file writes per hour: 50

**Circuit breaker**: Halt agent on repeated errors or limit breaches

---

## Incident Response

### If API Key Compromised

1. **Immediately**: Revoke key in provider dashboard
2. Rotate to new key
3. Update all references (repos, n8n workflows, env files)
4. Review logs for unauthorized usage
5. Assess impact (data accessed, costs incurred)
6. Document in incident log

### If Studio System Compromised

1. **Isolate**: Disconnect studio VM/VLAN from network
2. Rotate all credentials (GitHub tokens, API keys, passwords)
3. Review:
   - Git history for unauthorized commits
   - Recent GitHub Actions runs
   - n8n workflow execution logs
   - MCP server logs
4. Rebuild affected VMs from clean snapshots
5. Re-enable after full review

### If Personal/Corporate Data Leaked to LLM

1. **Document**: What data, which LLM provider, when
2. Assess risk:
   - PII? Report to GDPR/privacy team if required
   - Corporate? Report to IT/legal
   - Secrets? Follow key compromise process
3. Contact LLM provider to request data deletion (per their policies)
4. Update data handling procedures to prevent recurrence

---

## Compliance Notes

### GDPR (if handling EU data)

- Do NOT send PII to third-party LLMs without explicit consent and DPA
- Use local LLMs for any EU personal data processing
- Maintain data processing records

### Nestlé Policies (Corporate Constraint)

- **Zero** corporate data in studio systems
- Studio identity must be clearly distinct from corporate identity
- No studio work on corporate devices or networks

---

## Security Checklist

### Setup Phase

- [ ] Studio identity created (email, GitHub org)
- [ ] Separate browser profiles configured
- [ ] Studio VM deployed on isolated VLAN
- [ ] Firewall rules: studio ↛ corporate, studio ↛ personal
- [ ] Password manager with separate studio vault
- [ ] MFA enabled on all studio accounts
- [ ] GitHub fine-grained tokens created (not classic PATs)
- [ ] MCP servers sandboxed (filesystem, GitHub)
- [ ] API keys stored in vault/secrets manager, not code

### Operational Phase

- [ ] Data classification policy followed (no PII/corporate data to cloud LLMs)
- [ ] All AI actions logged centrally
- [ ] Rate limits configured on agents
- [ ] Weekly review of logs for anomalies
- [ ] Quarterly credential rotation

---

## Next Steps

See [HARDWARE.md](HARDWARE.md) for workstation and homelab specs.
