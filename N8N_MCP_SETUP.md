# n8n-mcp + n8n-skills Setup Guide

## Overview
**n8n-mcp** and **n8n-skills** are game-changing tools that make building n8n workflows 3-5x faster with Claude Code.

**Instead of:**
- Manually writing JSON workflow files
- Trial-and-error for node configurations
- Searching documentation for hours
- Debugging validation errors blindly

**You get:**
- Interactive workflow building with Claude
- Access to 1,084 documented nodes
- 2,646 real-world workflow examples
- Automatic validation and error detection
- Natural language → Production-ready workflows

**Time Savings: 30-50%** (from ~30 hours to ~15-20 hours for this project)

---

## Part 1: n8n-mcp Server Setup

### What is n8n-mcp?
A Model Context Protocol (MCP) server that gives Claude access to:
- 1,084 n8n nodes (537 core + 547 community, 301 verified)
- 99% node property coverage with schemas
- 87% documentation coverage from official n8n docs
- 265 AI-capable tool variants
- 2,646 real-world workflow configurations
- 2,709 workflow templates

### Prerequisites
- n8n instance running (cloud or self-hosted)
- Node.js installed (for npx)
- Claude Desktop or Claude Code
- n8n API credentials

---

## Step 1: Get n8n API Credentials

### For n8n Cloud:
1. Log in to [n8n.cloud](https://n8n.cloud/)
2. Click your profile (top right) → **Settings**
3. Go to **API** section
4. Click **Create API Key**
5. Name it: `Claude Code MCP Integration`
6. **Copy the API key** (looks like: `n8n_api_abc123...`)
7. **Copy your n8n instance URL** (format: `https://your-name.app.n8n.cloud`)

### For Self-Hosted n8n:
1. Access your n8n instance
2. Go to **Settings** → **API**
3. Click **Create API Key**
4. Name it: `Claude Code MCP Integration`
5. **Copy the API key**
6. Your instance URL is wherever you host n8n (e.g., `http://localhost:5678`)

**⚠️ Important:** Keep your API key private. It gives full access to your n8n workflows.

---

## Step 2: Install n8n-mcp Server

### Option A: npx Installation (Recommended - Easiest)

No installation needed! Run directly:

```bash
npx n8n-mcp
```

This will:
- Download the latest n8n-mcp server
- Run it in stdio mode (for Claude integration)
- Cache it for future use

**Pros:**
- No installation required
- Always uses latest version
- Works immediately

**Cons:**
- Slower first run (downloads package)
- Requires internet connection

### Option B: Docker (Isolated Environment)

```bash
# Pull the Docker image
docker pull ghcr.io/czlonkowski/n8n-mcp:latest

# Run it (configuration in next step)
```

**Pros:**
- Isolated environment
- Consistent across machines
- Easy updates

**Cons:**
- Requires Docker installed
- Slightly more complex setup

### Option C: Local Installation (For Development)

```bash
# Clone the repository
git clone https://github.com/czlonkowski/n8n-mcp.git
cd n8n-mcp

# Install dependencies
npm install

# Build
npm run build

# Note the installation path for configuration
pwd
```

**Pros:**
- Can modify source code
- No network dependency after install

**Cons:**
- Most complex setup
- Manual updates required

**For this project, use Option A (npx) - it's the easiest.**

---

## Step 3: Configure Claude Desktop/Code

### Find Your Claude Configuration File

**Mac:**
```
~/Library/Application Support/Claude/claude_desktop_config.json
```

**Windows:**
```
%APPDATA%\Claude\claude_desktop_config.json
```

**Linux:**
```
~/.config/Claude/claude_desktop_config.json
```

**Claude Code (VS Code Extension):**
```
~/.claude/.mcp.json
```

### Configuration for npx Installation

Edit the configuration file and add:

```json
{
  "mcpServers": {
    "n8n-mcp": {
      "command": "npx",
      "args": ["n8n-mcp"],
      "env": {
        "MCP_MODE": "stdio",
        "N8N_API_URL": "https://your-name.app.n8n.cloud",
        "N8N_API_KEY": "n8n_api_abc123..."
      }
    }
  }
}
```

**Replace:**
- `https://your-name.app.n8n.cloud` with your n8n instance URL
- `n8n_api_abc123...` with your actual API key

### Configuration for Docker Installation

```json
{
  "mcpServers": {
    "n8n-mcp": {
      "command": "docker",
      "args": [
        "run",
        "--rm",
        "-i",
        "-e", "MCP_MODE=stdio",
        "-e", "N8N_API_URL=https://your-name.app.n8n.cloud",
        "-e", "N8N_API_KEY=n8n_api_abc123...",
        "ghcr.io/czlonkowski/n8n-mcp:latest"
      ]
    }
  }
}
```

### Configuration for Local n8n (Localhost)

If running n8n locally:

```json
{
  "mcpServers": {
    "n8n-mcp": {
      "command": "npx",
      "args": ["n8n-mcp"],
      "env": {
        "MCP_MODE": "stdio",
        "N8N_API_URL": "http://host.docker.internal:5678",
        "N8N_API_KEY": "your-local-api-key",
        "N8N_WEBHOOK_SECURITY_MODE": "moderate"
      }
    }
  }
}
```

---

## Step 4: Restart Claude Desktop/Code

1. **Quit Claude** completely (not just close window)
   - Mac: Cmd+Q
   - Windows/Linux: File → Exit

2. **Reopen Claude**

3. **Verify MCP connection**:
   - You should see a small indicator showing MCP servers connected
   - Or check logs for "n8n-mcp server connected"

---

## Step 5: Test n8n-mcp Connection

Ask Claude any of these questions to test:

### Test 1: Search for a Node
```
You: "Find the Spotify node in n8n"
```

**Expected Response:**
Claude should return detailed information about the Spotify node, including:
- Node description
- Available operations (Get Track, Get Album, etc.)
- Required credentials
- Parameter schema

### Test 2: Search for Workflow Examples
```
You: "Show me workflow examples for API pagination in n8n"
```

**Expected Response:**
Claude should return examples from the 2,646 real-world workflows showing pagination patterns.

### Test 3: Get Node Configuration Help
```
You: "How do I configure the HTTP Request node to call the Spotify API?"
```

**Expected Response:**
Claude should provide step-by-step configuration guidance with proper authentication setup.

**If all tests work: ✅ n8n-mcp is ready!**

---

## Part 2: n8n-skills Installation

### What are n8n-skills?
7 specialized Claude Code skills that teach Claude how to build production-ready n8n workflows:

1. **n8n Expression Syntax** - Correct $json.body vs $json usage
2. **n8n MCP Tools Expert** (highest priority) - Tool selection & validation
3. **n8n Workflow Patterns** - Architectural best practices from 2,653+ templates
4. **n8n Validation Expert** - Interpret and fix validation errors
5. **n8n Node Configuration** - Operation-aware property dependencies
6. **n8n Code JavaScript** - Proper return formats (`[{json: {...}}]`)
7. **n8n Code Python** - When to use (rare - 5% use case)

### Installation

**In Claude Code:**
```bash
/plugin install czlonkowski/n8n-skills
```

**Expected Output:**
```
Installing plugin from czlonkowski/n8n-skills...
✅ Plugin installed successfully
7 skills added:
  - n8n Expression Syntax
  - n8n MCP Tools Expert
  - n8n Workflow Patterns
  - n8n Validation Expert
  - n8n Node Configuration
  - n8n Code JavaScript
  - n8n Code Python
```

### Verify Installation

Skills activate automatically when you work on n8n workflows. To test:

```
You: "I need to build an n8n workflow that fetches Spotify episodes"
```

**Expected Behavior:**
- Claude activates n8n MCP Tools Expert skill
- Searches for Spotify node using n8n-mcp
- Provides configuration guidance following best practices
- Validates your configuration automatically

---

## Troubleshooting

### Issue: "n8n-mcp server not connecting"

**Solution 1: Check Configuration**
- Verify `.mcp.json` syntax is valid JSON (no trailing commas!)
- Ensure `MCP_MODE` is set to `"stdio"`
- Check API URL has no trailing slash

**Solution 2: Check API Credentials**
```bash
# Test n8n API key manually
curl -X GET \
  'https://your-name.app.n8n.cloud/api/v1/workflows' \
  -H 'X-N8N-API-KEY: your-api-key'

# Should return list of workflows (or empty array)
# If error: API key is invalid
```

**Solution 3: Check Logs**
- Mac: `~/Library/Logs/Claude/mcp*.log`
- Windows: `%APPDATA%\Claude\logs\mcp*.log`
- Look for error messages

### Issue: "npx n8n-mcp command not found"

**Solution:**
1. Install Node.js: https://nodejs.org/
2. Verify installation: `node --version` (should show v16+)
3. Retry: `npx n8n-mcp`

### Issue: "n8n-skills not activating"

**Solution:**
1. Verify installation: `/plugin list`
2. Look for `czlonkowski/n8n-skills` in the list
3. If not there: Reinstall with `/plugin install czlonkowski/n8n-skills`
4. Skills activate automatically - no manual trigger needed

### Issue: "Permission denied" errors

**Solution:**
- Mac/Linux: Check file permissions on config file
  ```bash
  chmod 644 ~/.claude/.mcp.json
  ```
- Make sure Claude app has necessary permissions

### Issue: "Rate limiting" or "Too many requests"

**Solution:**
- n8n-mcp uses your n8n API quota
- Default limits are generous (shouldn't hit them)
- If you do: Wait 1 minute and retry
- Check n8n API settings for your tier limits

---

## Optional: Disable Telemetry

n8n-mcp has optional telemetry to improve the tool. To disable:

Add to your environment variables:
```json
{
  "env": {
    "N8N_MCP_TELEMETRY_DISABLED": "true"
  }
}
```

---

## Advanced Configuration

### Use Hosted Service (Alternative to Self-Hosting)

Instead of running n8n-mcp locally, use the hosted service:

1. Go to [dashboard.n8n-mcp.com](https://dashboard.n8n-mcp.com)
2. Sign up for free tier (100 tool calls/day)
3. Get your API token
4. Configure Claude:

```json
{
  "mcpServers": {
    "n8n-mcp": {
      "command": "npx",
      "args": ["n8n-mcp"],
      "env": {
        "N8N_MCP_API_TOKEN": "your-hosted-api-token"
      }
    }
  }
}
```

**Pros:**
- No local server needed
- Always up-to-date
- Managed infrastructure

**Cons:**
- Requires internet
- Free tier limits (100 calls/day)
- Can't manage workflows via API (read-only mode)

### Deploy to Railway (One-Click Deployment)

For team use, deploy to Railway cloud:

1. Visit https://github.com/czlonkowski/n8n-mcp
2. Click "Deploy on Railway" button
3. Set environment variables:
   - `N8N_API_URL`
   - `N8N_API_KEY`
4. Get Railway deployment URL
5. Configure Claude to use Railway endpoint

---

## Next Steps

✅ n8n-mcp server installed and configured
✅ n8n-skills installed
✅ Connection tested successfully

**You're ready to build n8n workflows with Claude!**

**Continue to:** Main README.md → Week 2: Build LaWayra Guest Extraction workflow

---

## Resources

- **n8n-mcp GitHub**: https://github.com/czlonkowski/n8n-mcp
- **n8n-skills GitHub**: https://github.com/czlonkowski/n8n-skills
- **n8n API Docs**: https://docs.n8n.io/api/
- **MCP Protocol**: https://modelcontextprotocol.io/

## Quick Reference

**Test n8n-mcp:**
```
Ask Claude: "Find the HTTP Request node in n8n"
```

**Search workflow examples:**
```
Ask Claude: "Show me n8n workflows that use Spotify API"
```

**Get configuration help:**
```
Ask Claude: "How do I configure authentication for Listen Notes API in n8n?"
```

**Validate a workflow:**
```
Ask Claude: "Check this n8n workflow for common mistakes"
```

---

Last Updated: 2026-02-15
