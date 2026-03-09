# Model Context Protocol (MCP) Integration Guide

How to use MCP to connect Claude and Claude Code to external tools.

---

## What is MCP?

Model Context Protocol is an open standard that lets AI assistants (like Claude) safely interact with external tools and data sources through a standardized interface.

**Benefits**:
- Uniform API across tools (filesystem, GitHub, databases, APIs)
- Explicit permissions and sandboxing
- Stateful connections (tools can maintain context)
- Easier to build and share custom tools

**Official docs**: https://modelcontextprotocol.io

---

## Core Concepts

### MCP Servers

Background processes that expose tools to AI assistants.

**Examples**:
- `@modelcontextprotocol/server-filesystem`: File operations
- `@modelcontextprotocol/server-github`: GitHub API
- Custom servers: Database queries, API calls, hardware control

### MCP Clients

AI assistants that call MCP servers.

**Examples**:
- Claude Desktop
- Claude Code
- Custom applications using MCP SDK

### Tools

Functions exposed by MCP servers that the AI can call.

**Example tools** (filesystem server):
- `read_file(path)`
- `write_file(path, content)`
- `list_directory(path)`
- `search_files(pattern)`

---

## Setting Up MCP

### Prerequisites

- Node.js 18+ installed
- Claude Desktop or Claude Code
- npx available

### Configuration File

**Location**:
- macOS: `~/Library/Application Support/Claude/claude_desktop_config.json`
- Windows: `%APPDATA%\Claude\claude_desktop_config.json`
- Linux: `~/.config/Claude/claude_desktop_config.json`

**Format**:

```json
{
  "mcpServers": {
    "server-name": {
      "command": "npx",
      "args": ["-y", "package-name", "...args"],
      "env": {
        "KEY": "value"
      }
    }
  }
}
```

---

## Built-in MCP Servers

### Filesystem Server

**Purpose**: Read and write files in a sandboxed directory.

**Install**:
```bash
npm install -g @modelcontextprotocol/server-filesystem
```

**Configure**:
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

**Tools provided**:
- `read_file`: Read file contents
- `read_multiple_files`: Batch read
- `write_file`: Create or overwrite file
- `edit_file`: Apply diff-style edits
- `create_directory`: Make directories
- `list_directory`: List files and subdirectories
- `move_file`: Rename or move
- `search_files`: Grep-style search
- `get_file_info`: Stats (size, modified date)

**Security**: Only accesses the specified directory tree.

**Example prompt**:
> "List all Python files in my workspace, then read the main.py file"

### GitHub Server

**Purpose**: Interact with GitHub repositories, issues, and pull requests.

**Install**:
```bash
npm install -g @modelcontextprotocol/server-github
```

**Configure**:
```json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_TOKEN": "ghp_your_fine_grained_token_here"
      }
    }
  }
}
```

**Tools provided**:
- `create_repository`: New repo
- `create_or_update_file`: Commit file changes
- `search_repositories`: Find repos
- `create_issue`: New issue
- `create_pull_request`: New PR
- `fork_repository`: Fork a repo
- `create_branch`: New branch
- `list_issues`: Query issues
- `update_issue`: Edit issue
- `add_issue_comment`: Comment on issue
- `search_code`: Search code across GitHub
- `get_file_contents`: Read files from repos

**Security**: Use fine-grained tokens scoped to specific orgs/repos.

**Example prompt**:
> "Create a new issue in zebadee2kk/HeliOS-Studio titled 'Add authentication' with the label 'enhancement'"

---

## Custom MCP Servers

### Example: n8n Workflow Trigger

**Goal**: Let Claude trigger n8n workflows.

**Implementation** (`mcp-servers/n8n/index.js`):

```javascript
const { MCPServer } = require('@modelcontextprotocol/sdk');
const axios = require('axios');

const server = new MCPServer({
  name: 'n8n',
  version: '1.0.0',
});

const N8N_URL = process.env.N8N_URL || 'http://localhost:5678';

server.addTool({
  name: 'trigger_workflow',
  description: 'Trigger an n8n workflow by webhook name',
  parameters: {
    type: 'object',
    properties: {
      webhook: {
        type: 'string',
        description: 'Webhook name (e.g., "process-research")',
      },
      payload: {
        type: 'object',
        description: 'Data to send to workflow',
      },
    },
    required: ['webhook'],
  },
  async handler({ webhook, payload = {} }) {
    try {
      const response = await axios.post(
        `${N8N_URL}/webhook/${webhook}`,
        payload
      );
      return {
        success: true,
        data: response.data,
      };
    } catch (error) {
      return {
        success: false,
        error: error.message,
      };
    }
  },
});

server.addTool({
  name: 'get_workflow_status',
  description: 'Check the status of recent n8n workflow executions',
  parameters: {
    type: 'object',
    properties: {
      webhook: {
        type: 'string',
        description: 'Webhook name',
      },
      limit: {
        type: 'number',
        description: 'Max number of executions to return',
        default: 10,
      },
    },
    required: ['webhook'],
  },
  async handler({ webhook, limit = 10 }) {
    // Query n8n API for recent executions
    // Simplified - actual implementation would use n8n REST API
    return {
      executions: [
        { id: 1, status: 'success', started: '2026-03-09T08:00:00Z' },
        { id: 2, status: 'running', started: '2026-03-09T09:00:00Z' },
      ],
    };
  },
});

server.start();
```

**Package.json**:
```json
{
  "name": "mcp-server-n8n",
  "version": "1.0.0",
  "main": "index.js",
  "dependencies": {
    "@modelcontextprotocol/sdk": "^0.1.0",
    "axios": "^1.6.0"
  },
  "bin": {
    "mcp-server-n8n": "./index.js"
  }
}
```

**Configure in Claude**:
```json
{
  "mcpServers": {
    "n8n": {
      "command": "node",
      "args": ["/path/to/mcp-servers/n8n/index.js"],
      "env": {
        "N8N_URL": "http://10.100.0.20:5678"
      }
    }
  }
}
```

**Usage**:
> "Trigger the 'process-research' workflow with payload {topic: 'MCP security'}"

### Example: Ollama Local LLM

**Goal**: Let Claude use local models for cheap/private subtasks.

**Implementation** (`mcp-servers/ollama/index.js`):

```javascript
const { MCPServer } = require('@modelcontextprotocol/sdk');
const axios = require('axios');

const server = new MCPServer({
  name: 'ollama',
  version: '1.0.0',
});

const OLLAMA_URL = process.env.OLLAMA_URL || 'http://localhost:11434';

server.addTool({
  name: 'local_completion',
  description: 'Generate text using a local Ollama model',
  parameters: {
    type: 'object',
    properties: {
      model: {
        type: 'string',
        description: 'Model name (e.g., llama3, qwen3, phi3)',
      },
      prompt: {
        type: 'string',
        description: 'Prompt text',
      },
      max_tokens: {
        type: 'number',
        default: 500,
      },
    },
    required: ['model', 'prompt'],
  },
  async handler({ model, prompt, max_tokens = 500 }) {
    const response = await axios.post(`${OLLAMA_URL}/api/generate`, {
      model,
      prompt,
      options: {
        num_predict: max_tokens,
      },
    });
    return {
      text: response.data.response,
      model,
      tokens: response.data.eval_count,
    };
  },
});

server.addTool({
  name: 'local_embeddings',
  description: 'Generate embeddings using a local Ollama model',
  parameters: {
    type: 'object',
    properties: {
      model: {
        type: 'string',
        description: 'Embedding model (e.g., mxbai-embed-large)',
      },
      texts: {
        type: 'array',
        items: { type: 'string' },
        description: 'Array of texts to embed',
      },
    },
    required: ['model', 'texts'],
  },
  async handler({ model, texts }) {
    const embeddings = await Promise.all(
      texts.map(async (text) => {
        const response = await axios.post(`${OLLAMA_URL}/api/embeddings`, {
          model,
          prompt: text,
        });
        return response.data.embedding;
      })
    );
    return { embeddings };
  },
});

server.start();
```

**Usage**:
> "Use the local llama3 model to summarize this log file (use the filesystem MCP to read it first)"

---

## Security Best Practices

### Principle of Least Privilege

- **Filesystem**: Only grant access to specific workspace directories, not `/` or `~`
- **GitHub**: Use fine-grained tokens scoped to specific repos, not classic PATs
- **Custom servers**: Only expose necessary tools; validate all inputs

### Sandboxing

**Run MCP servers in containers**:

```yaml
services:
  mcp-filesystem:
    image: node:18
    command: npx -y @modelcontextprotocol/server-filesystem /workspace
    volumes:
      - /home/user/workspace:/workspace:ro  # Read-only
    networks:
      - mcp_net
    read_only: true
    security_opt:
      - no-new-privileges:true
```

### Input Validation

For custom MCP servers, validate all parameters:

```javascript
server.addTool({
  name: 'delete_file',
  async handler({ path }) {
    // Validate path is within allowed directory
    const resolved = require('path').resolve(path);
    if (!resolved.startsWith('/workspace')) {
      throw new Error('Access denied: path outside workspace');
    }
    
    // Validate no directory traversal
    if (path.includes('..')) {
      throw new Error('Invalid path: directory traversal not allowed');
    }
    
    // Proceed with deletion
    await fs.unlink(resolved);
    return { success: true };
  },
});
```

### Audit Logging

Log all MCP tool calls:

```javascript
const logger = require('winston').createLogger({...});

server.addTool({
  name: 'sensitive_operation',
  async handler(params) {
    logger.info('MCP tool called', {
      tool: 'sensitive_operation',
      params,
      timestamp: new Date().toISOString(),
    });
    // ... actual implementation
  },
});
```

Ship logs to central system (ELK, Loki, etc.) for monitoring.

---

## Troubleshooting

### MCP Server Not Connecting

**Symptoms**: Claude says "Tool not available" or MCP server doesn't appear in settings.

**Debug steps**:

1. **Check Claude logs**: `Help > Show Logs` in Claude Desktop
2. **Verify config syntax**: Validate JSON in config file
3. **Test MCP server manually**:

```bash
npx -y @modelcontextprotocol/server-filesystem /tmp
# Should output: MCP server listening...
```

4. **Check permissions**: Ensure file paths are readable
5. **Restart Claude Desktop** after config changes

### Tool Calls Failing

**Symptoms**: Claude attempts to call tool but gets errors.

**Debug**:

1. **Check MCP server logs**: `journalctl -u mcp-server` (if systemd service)
2. **Verify environment variables**: GitHub token set correctly?
3. **Test tool directly**: Use MCP SDK to call tool without Claude
4. **Check rate limits**: GitHub API, etc.

### Performance Issues

**Symptoms**: Tool calls are slow.

**Fixes**:

- **Network latency**: Run MCP servers on same machine as Claude
- **Heavy operations**: Add caching to custom MCP servers
- **Large responses**: Paginate or summarize data before returning

---

## Advanced Patterns

### Chaining MCP Tools

Claude can orchestrate multiple tool calls:

**Prompt**:
> "Search my GitHub repos for 'security', find the top 3, and for each one read the README file"

**Claude's plan**:
1. Call `github:search_repositories` with query "security"
2. For each of top 3 results:
   - Call `github:get_file_contents` with path "README.md"
3. Summarize findings

### Conditional Tool Use

**Prompt**:
> "If the latest experiment in my repo failed, trigger the 'retry-experiment' n8n workflow"

**Claude's plan**:
1. Call `github:get_file_contents` for `experiments/latest.md`
2. Parse status
3. If status == 'failed':
   - Call `n8n:trigger_workflow` with webhook "retry-experiment"

### Human-in-the-Loop

**Prompt**:
> "Draft a new GitHub issue for this bug, then ask me to review before creating it"

**Claude's plan**:
1. Generate issue title, body, labels
2. Show draft to user
3. Wait for approval
4. If approved, call `github:create_issue`

---

## Next Steps

See [LOCAL_LLMS.md](LOCAL_LLMS.md) for local model selection and optimization.
