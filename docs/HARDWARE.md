# HeliOS Studio Hardware Recommendations

Optimal hardware configuration for the full stack.

---

## Workstation Specification

### Primary Development Machine

This is where you spend most of your time: Cursor, Claude Desktop, browsers.

**CPU**
- **Minimum**: 12-core (Ryzen 5900X, Intel i7-12700)
- **Recommended**: 16-24 core (Ryzen 7950X, Intel i9-13900K, Apple M3 Max)
- **Why**: Parallel compilation, multiple Docker containers, local model inference

**RAM**
- **Minimum**: 32GB
- **Recommended**: 64GB
- **Ideal**: 128GB (for running 13B+ models locally)
- **Why**: Browser tabs, IDE, Docker, local LLMs, multiple VMs

**GPU** (if running local LLMs on workstation)
- **Minimum**: 8GB VRAM (RTX 3060, RTX 4060)
- **Recommended**: 16GB VRAM (RTX 4060 Ti 16GB, RTX 4070)
- **Ideal**: 24GB VRAM (RTX 4090, RTX 6000 Ada)
- **Why**: 7-8B models need ~8GB; 13B models need ~16GB; 30B+ models need 24GB+

**Storage**
- **Minimum**: 1TB NVMe SSD
- **Recommended**: 2TB NVMe SSD (Gen4)
- **Ideal**: 2TB primary + 4TB secondary SSD
- **Why**: Repos, Docker images, model weights, databases

**Network**
- **Minimum**: 1 Gbps Ethernet
- **Recommended**: 2.5 Gbps Ethernet or 10 Gbps fiber
- **Why**: Fast artifact transfer to homelab, Docker image pushes

---

## Homelab Configuration

### Existing Setup

You already have:
- 2x Proxmox hosts
- QNAP TS-253 Pro NAS
- GPUs: NVIDIA T600, RTX 4000 Quadro
- ~11 LXC containers, 3+ VMs

### Recommended VM Allocation

#### AI Core VM

**Purpose**: Ollama, vector databases, local model serving

**Specs**:
- **CPU**: 8 cores
- **RAM**: 32GB
- **GPU**: RTX 4000 Quadro (passthrough)
- **Storage**: 500GB SSD (for model weights)
- **OS**: Ubuntu 22.04 LTS

**Services**:
- Ollama (port 11434)
- Qdrant or Weaviate (vector DB)
- Optional: vLLM or Text Generation Inference for production serving

#### Automation Hub VM

**Purpose**: n8n, monitoring, observability

**Specs**:
- **CPU**: 4 cores
- **RAM**: 16GB
- **Storage**: 200GB SSD
- **OS**: Ubuntu 22.04 LTS

**Services**:
- n8n (port 5678)
- Prometheus (port 9090)
- Grafana (port 3000)
- Uptime Kuma (port 3001)

#### GitHub Runners VM(s)

**Purpose**: Self-hosted GitHub Actions runners

**Specs** (per runner):
- **CPU**: 4 cores
- **RAM**: 8GB
- **Storage**: 100GB SSD
- **OS**: Ubuntu 22.04 LTS

**Why self-hosted**:
- Faster builds (local network)
- Access to homelab services (Ollama, databases)
- No queue time
- Free (vs GitHub-hosted minutes)

**Setup**:
```bash
# On runner VM
mkdir actions-runner && cd actions-runner
curl -o actions-runner-linux-x64-2.311.0.tar.gz -L https://github.com/actions/runner/releases/download/v2.311.0/actions-runner-linux-x64-2.311.0.tar.gz
tar xzf ./actions-runner-linux-x64-2.311.0.tar.gz
./config.sh --url https://github.com/your-studio --token <TOKEN>
sudo ./svc.sh install
sudo ./svc.sh start
```

#### Studio Workstation VM (Optional)

**Purpose**: Isolated development environment

**Specs**:
- **CPU**: 8 cores
- **RAM**: 32GB
- **GPU**: T600 (passthrough) or vGPU slice
- **Storage**: 500GB SSD
- **OS**: Ubuntu 22.04 Desktop or Windows 11

**Services**:
- Cursor / VS Code
- Claude Desktop
- Browsers (separate profiles)
- Docker Desktop

**Why**: 
- Complete isolation from personal/corporate machines
- Can be snapshotted and rolled back
- Accessible remotely via RDP/VNC

---

## GPU Strategy

### Current GPUs

| GPU | VRAM | CUDA Cores | Best For |
|-----|------|------------|----------|
| NVIDIA T600 | 4GB | 896 | Small models (3-4B), CPU offload, testing |
| RTX 4000 Quadro | 8GB | 6144 | 7-8B models, stable diffusion, encoding |

### Model Size → VRAM Requirements

| Model Size | VRAM (FP16) | VRAM (4-bit quant) | Fits On |
|------------|-------------|---------------------|----------|
| 3-4B (Phi-3, Qwen3-4B) | ~8GB | ~3GB | T600 (4-bit), RTX 4000 |
| 7-8B (Llama 3, Qwen3-7B) | ~16GB | ~6GB | RTX 4000 (4-bit) |
| 13-14B (Llama 3-13B) | ~26GB | ~10GB | Not feasible on current setup |
| 30B+ (Qwen3-30B) | ~60GB | ~20GB | Not feasible on current setup |

### Recommendations

**For current setup**:
- **RTX 4000**: Run 7-8B models in 4-bit quantization (good quality, ~20-30 tok/s)
- **T600**: Run 3-4B models (Phi-3 Mini) for fast utility tasks

**Upgrade path** (if budget allows):
- Add **RTX 4060 Ti 16GB** ($500): Unlocks 13B models in 4-bit (~15 tok/s)
- Add **RTX 4090 24GB** ($1600): Unlocks 30B+ models, faster inference
- Alternative: **Used RTX A6000 48GB** (~$2500): Best for large models and batching

---

## Network Architecture

### VLAN Design

```
┌────────────────────────────┐
│     VLAN 1: Management     │  10.1.0.0/24
│  Proxmox hosts, QNAP, etc. │  • Admin access only
└────────────────────────────┘

┌────────────────────────────┐
│   VLAN 10: Corporate      │  10.10.0.0/24
│   Work devices/laptop     │  • Isolated from studio
└────────────────────────────┘

┌────────────────────────────┐
│   VLAN 20: Personal       │  10.20.0.0/24
│   Personal devices        │  • Home network
└────────────────────────────┘

┌────────────────────────────┐
│  VLAN 100: Studio        │  10.100.0.0/24
│  • Studio workstation VM  │  • Isolated from corp/personal
│  • AI Core VM (Ollama)    │  • Internet access allowed
│  • Automation Hub (n8n)   │  • GitHub access
│  • GitHub Runners         │
└────────────────────────────┘
```

### Firewall Rules

```
# Allow studio to internet
VLAN 100 → WAN: ALLOW

# Block studio from corporate and personal
VLAN 100 → VLAN 10: DENY
VLAN 100 → VLAN 20: DENY

# Allow management to all (for admin)
VLAN 1 → *: ALLOW

# Block corporate from studio
VLAN 10 → VLAN 100: DENY
```

---

## Cost Summary

### Current Investment

- Proxmox hosts: Already owned
- QNAP NAS: Already owned
- T600, RTX 4000: Already owned

**New costs** (minimal):
- Studio domain: $10-20/year
- VPS (optional): $5-20/month

**Total new**: ~$200/year

### Optional Upgrades

| Item | Cost | Benefit |
|------|------|----------|
| RTX 4060 Ti 16GB | $500 | Run 13B models locally |
| 64GB RAM upgrade | $200 | More headroom for containers |
| 2TB NVMe SSD | $150 | More storage for models/data |
| 10 Gbps NIC + switch | $300 | Faster homelab transfers |
| **Total** | **$1150** | Significantly better local AI capabilities |

---

## Storage Strategy

### Workstation

- **SSD 1** (2TB NVMe): OS, applications, active repos
- **SSD 2** (4TB SATA): Docker volumes, databases, archives

### Homelab

- **QNAP NAS**: Long-term storage
  - Model weights (Ollama library)
  - Backups (GitHub repos, databases)
  - Artifacts (experiment results, logs)
  - Media (NotebookLM exports, diagrams)

- **Per VM**: Local SSD for performance
  - AI Core: 500GB for active models
  - Automation Hub: 200GB for databases
  - Runners: 100GB for build caches

### Backup Strategy

1. **GitHub**: Source of truth for all code and docs
2. **QNAP**: Nightly snapshots of VMs, databases, and model weights
3. **Cloud backup**: Weekly encrypted backup of QNAP to Backblaze B2 or similar

---

## Power and Cooling

### Power Consumption

**Workstation** (under load):
- CPU: 120-200W
- GPU: 100-200W
- Total: ~300-500W

**Homelab** (24/7):
- 2x Proxmox hosts: ~200W each
- QNAP NAS: ~50W
- Network gear: ~50W
- Total: ~500W continuous = ~360 kWh/month

**Cost** (at $0.15/kWh): ~$54/month

### Cooling

- Ensure homelab has adequate ventilation
- Consider rack exhaust fan if in enclosed cabinet
- Monitor temps via Proxmox + Grafana

---

## Monitoring

### Hardware Monitoring

**Prometheus exporters**:
- `node_exporter`: CPU, RAM, disk, network
- `nvidia_gpu_exporter`: GPU utilization, VRAM, temp
- `smartctl_exporter`: Disk health (SMART data)

**Grafana dashboards**:
- Homelab overview (all hosts)
- Per-VM resource usage
- GPU utilization over time
- Storage capacity trends

**Alerting**:
- CPU > 90% for 10 minutes
- RAM > 95%
- Disk > 85% full
- GPU temp > 80°C
- Any disk SMART errors

---

## Next Steps

See [MCP.md](MCP.md) for Model Context Protocol integration details.
