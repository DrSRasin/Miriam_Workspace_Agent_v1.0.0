# Dr. Miriam Workspace Agent: Complete Setup & Integration Guide

**Agent Name:** Dr. Miriam Workspace Agent  
**Version:** 1.0.0  
**Status:** Evidence-first, declarative agent with Tavily web research + GitHub integration  
**Last Updated:** 2026-09-03  

---

## Table of Contents

1. [Quick Start](#quick-start)
2. [Tavily Integration](#tavily-integration)
3. [GitHub Integration](#github-integration)
4. [Project Structure](#project-structure)
5. [Configuration Files](#configuration-files)
6. [Environment Setup](#environment-setup)
7. [Running the Agent](#running-the-agent)
8. [ngrok Remote Access](#ngrok-remote-access)
9. [Troubleshooting](#troubleshooting)

---

## Quick Start

### Prerequisites

- macOS/Linux environment (you're on macOS)
- Node.js 18+ and npm
- Microsoft 365 Agents Toolkit CLI (`atk`)
- Tavily CLI (`tvly`) – already installed and authenticated ✅
- GitHub Personal Access Token (PAT) for private repo access

### Installation Steps (5 minutes)

```bash
# 1. Clone this repository
git clone https://github.com/DrSRasin/Miriam_Workspace_Agent_v1.0.0.git
cd Miriam_Workspace_Agent_v1.0.0

# 2. Install dependencies
npm install

# 3. Set up environment variables
cp .env.example .env
# Edit .env with your API keys (see Environment Setup section below)

# 4. Verify Tavily authentication
tvly --status

# 5. Verify M365 Agents Toolkit
atk doctor

# 6. Start the agent
npm run dev
```

---

## Tavily Integration

### Current Status ✅

Tavily CLI is **already authenticated** at:
```
/Users/macbookpro2023/.tavily/config.json
```

Authentication method: **API key**  
API key location: `~/.tavily/config.json` (managed by Tavily CLI)

### Available Tavily Commands

Dr. Miriam has access to five core Tavily commands:

| Command | Use Case | Speed | Output |
|---------|----------|-------|--------|
| `tvly search` | Find pages on a topic | Fast | JSON with snippets & sources |
| `tvly extract` | Get content from a URL | Medium | Markdown/text content |
| `tvly crawl` | Bulk extract from a site | Slow | Multiple markdown files |
| `tvly map` | Discover URLs on a site | Fast | URL inventory only |
| `tvly research` | Deep multi-source analysis | Slowest (30-120s) | Synthesized report with citations |

### Tavily Workflow (Escalation Pattern)

1. **Search first** → Find relevant sources
2. **Extract if needed** → Get full content from specific URL
3. **Map if large site** → Discover all pages before crawling
4. **Crawl for bulk** → Extract entire site sections
5. **Research for depth** → Comprehensive synthesis with citations

### Example Commands

```bash
# Basic search
tvly search "latest AI developments" --json

# Advanced search with filters
tvly search "machine learning research" --depth advanced --max-results 10 --time-range month --json

# Extract from specific URL
tvly extract "https://example.com/article" --json

# Deep research (takes 30-120 seconds)
tvly research "climate change policy 2026" --json --model pro

# Map a website
tvly map "https://docs.example.com" --json

# Crawl and save locally
tvly crawl "https://docs.example.com/guides" --output-dir ./research-output/
```

### Tavily CLI Reference

Full documentation: `tvly --help`

Common options:
- `--json` – Output structured JSON (required for agent workflows)
- `--depth {ultra-fast|fast|basic|advanced}` – Search depth (default: basic)
- `--max-results N` – Limit results (0-20, default: 5)
- `--time-range {day|week|month|year}` – Filter by date
- `--include-domains example.com,other.com` – Focus on trusted sources
- `--exclude-domains spam.com` – Block specific domains
- `-o FILE` – Save output to file

---

## GitHub Integration

### Setup: GitHub Personal Access Token

Dr. Miriam needs read/write access to your private GitHub repositories.

#### Step 1: Generate GitHub PAT

```bash
# Visit: https://github.com/settings/tokens/new

# Required scopes:
✓ repo (full control of private and public repos)
✓ read:org
✓ admin:repo_hook (optional, for webhooks)
✓ workflow (optional, for GitHub Actions)

# Name it: "Dr Miriam Agent PAT"
# Expiration: 90 days (recommended for security)
# Save the token string
```

#### Step 2: Store Token Securely

**Never commit the token to GitHub!** Use environment variables:

```bash
# Add to .env file
echo "GITHUB_TOKEN=ghp_YOUR_TOKEN_HERE" >> .env

# Or export directly in terminal
export GITHUB_TOKEN=ghp_YOUR_TOKEN_HERE
```

#### Step 3: Verify Access

```bash
# Test GitHub CLI authentication
gh auth status

# Test API access
gh repo list --private

# Test with curl (if not using GitHub CLI)
curl -H "Authorization: token $GITHUB_TOKEN" \
  https://api.github.com/user/repos?type=private
```

### GitHub Capabilities for Dr. Miriam

With proper GitHub integration, Dr. Miriam can:

#### Read Operations
- ✅ List all repositories (public + private)
- ✅ Read repository contents and structure
- ✅ Fetch file contents and file history
- ✅ List issues, pull requests, and discussions
- ✅ Read GitHub Actions workflows
- ✅ Access repository settings and metadata

#### Write Operations
- ✅ Create and update files in repositories
- ✅ Create branches and manage refs
- ✅ Create and update issues
- ✅ Create pull requests and manage reviews
- ✅ Commit changes with proper messages
- ✅ Manage repository labels and milestones

### GitHub Tool Commands in Agent YAML

In your `m365agents.yml`, Dr. Miriam will have access to:

```yaml
tools:
  # GitHub repository operations
  - id: github-list-repos
    type: bash
    command: gh repo list --private --json nameWithOwner,url
    
  - id: github-read-file
    type: bash
    command: gh repo view {owner}/{repo} --json description,url
    
  - id: github-create-branch
    type: bash
    command: gh api repos/{owner}/{repo}/git/refs -f ref=refs/heads/{branch} -f sha={sha}
    
  - id: github-create-issue
    type: bash
    command: gh issue create -R {owner}/{repo} -t "{title}" -b "{body}" -l {labels}
    
  - id: github-search-code
    type: bash
    command: gh search code "{query}" --repo {owner}/{repo} --json path,name,repository
```

---

## Project Structure

```
Miriam_Workspace_Agent_v1.0.0/
│
├── README.md                      ← Original Microsoft 365 Agents info
├── TAVILY_SETUP.md               ← This file - Complete setup guide
├── GITHUB_INTEGRATION.md          ← GitHub access & API reference
├── .env.example                   ← Template for environment variables
├── .env                           ← (Local only, .gitignore'd) Your secrets
├── .gitignore                     ← Standard Node.js + secrets
│
├── manifest.json                  ← Microsoft Teams manifest
├── declarativeAgent.json          ← Agent behavior & instructions
│
├── m365agents.yml                 ← Main M365 Agents Toolkit config
├── m365agents.local.yml           ← Local dev overrides (ngrok, debug)
│
├── package.json                   ← Node.js dependencies
├── tsconfig.json                  ← TypeScript configuration
│
├── src/
│   ├── index.ts                   ← Agent entry point
│   ├── tools/
│   │   ├── tavily.ts             ← Tavily CLI wrapper
│   │   └── github.ts             ← GitHub API wrapper
│   └── config/
│       └── environment.ts         ← Environment variable loader
│
├── config/
│   ├── tavily-tools.json          ← Tavily tool definitions
│   ├── github-tools.json          ← GitHub tool definitions
│   └── m365-permissions.json      ← M365 OAuth scopes
│
└── docs/
    ├── WORKFLOW.md                ← Agent decision workflow
    └── TROUBLESHOOTING.md         ← Common issues & fixes
```

---

## Configuration Files

### 1. m365agents.yml (Main Configuration)

```yaml
version: 1.0
name: Miriam Workspace Agent
description: Evidence-first workspace and web research assistant with GitHub integration

# Tool definitions
tools:
  # ========== TAVILY WEB RESEARCH ==========
  - id: tavily-search
    type: bash
    description: Search the web with LLM-optimized results
    command: tvly search "{query}" --json --depth advanced
    timeout: 30000
    environment:
      TAVILY_API_KEY: ${TAVILY_API_KEY}
    
  - id: tavily-extract
    type: bash
    description: Extract clean markdown content from a specific URL
    command: tvly extract "{url}" --json
    timeout: 20000
    environment:
      TAVILY_API_KEY: ${TAVILY_API_KEY}
    
  - id: tavily-research
    type: bash
    description: Comprehensive AI-powered research with citations
    command: tvly research "{topic}" --json --model pro
    timeout: 120000
    environment:
      TAVILY_API_KEY: ${TAVILY_API_KEY}

  # ========== GITHUB REPOSITORY ACCESS ==========
  - id: github-list-repos
    type: bash
    description: List all your GitHub repositories (public and private)
    command: gh repo list --private --limit 100 --json nameWithOwner,description,isPrivate
    timeout: 15000
    environment:
      GITHUB_TOKEN: ${GITHUB_TOKEN}
    
  - id: github-read-file
    type: bash
    description: Read a file from any of your GitHub repositories
    command: gh api repos/{owner}/{repo}/contents/{path}
    timeout: 10000
    environment:
      GITHUB_TOKEN: ${GITHUB_TOKEN}
    
  - id: github-create-issue
    type: bash
    description: Create a new issue in a repository
    command: gh issue create -R {owner}/{repo} -t "{title}" -b "{body}"
    timeout: 10000
    environment:
      GITHUB_TOKEN: ${GITHUB_TOKEN}

# Agent capabilities
capabilities:
  reasoning: true
  planning: true
  tool_use: true
  streaming: true
  web_search: true
  code_analysis: true
  evidence_grading: true

# M365 OAuth Scopes
permissions:
  - User.Read
  - Mail.Read
  - Calendar.Read
  - Files.Read.All
```

### 2. m365agents.local.yml (Development Overrides)

```yaml
version: 1.0
name: Miriam Workspace Agent (Local Dev)

# Development endpoints
endpoints:
  base_url: ${NGROK_URL}  # e.g., https://abc123.ngrok.io

tools:
  - id: tavily-search
    type: bash
    command: tvly search "{query}" --json --depth basic
    timeout: 30000
    environment:
      TAVILY_API_KEY: ${TAVILY_API_KEY}
      DEBUG: "true"
      LOG_LEVEL: "debug"

  - id: github-list-repos
    type: bash
    command: gh repo list --private --json nameWithOwner
    timeout: 15000
    environment:
      GITHUB_TOKEN: ${GITHUB_TOKEN}
      DEBUG: "true"

capabilities:
  debug_mode: true
  verbose_logging: true
```

### 3. .env (Secrets Template)

```bash
# ========== TAVILY WEB RESEARCH ==========
# Get from: https://tavily.com
TAVILY_API_KEY=tvly-YOUR_KEY_HERE

# ========== GITHUB ACCESS ==========
# Get from: https://github.com/settings/tokens/new
# Scopes: repo, read:org, workflow
GITHUB_TOKEN=ghp_YOUR_PAT_HERE

# ========== NGROK (Remote Access via Web) ==========
# Get from: https://dashboard.ngrok.com/auth/your-authtokens
NGROK_AUTHTOKEN=your_ngrok_authtoken
NGROK_TUNNEL_URL=https://your-ngrok-url.ngrok.io

# ========== MICROSOFT 365 (Optional - for native M365 integration) ==========
CLIENT_ID=your_m365_client_id
CLIENT_SECRET=your_m365_client_secret
TENANT_ID=your_tenant_id

# ========== LOGGING & DEBUG ==========
LOG_LEVEL=info
DEBUG=false
```

### 4. package.json

```json
{
  "name": "miriam-workspace-agent",
  "version": "1.0.0",
  "description": "Dr. Miriam: Evidence-first workspace agent with Tavily & GitHub",
  "main": "src/index.ts",
  "scripts": {
    "start": "node src/index.ts",
    "dev": "atk preview --debug",
    "dev:local": "atk preview --m365agents-file m365agents.local.yml --debug",
    "build": "atk package",
    "provision": "atk provision",
    "deploy": "atk deploy",
    "validate": "atk validate",
    "test": "npm run lint && atk validate",
    "lint": "eslint src/ --ext .ts",
    "clean": "rm -rf dist/ node_modules/"
  },
  "keywords": [
    "m365-agents",
    "copilot",
    "declarative-agent",
    "tavily",
    "github"
  ],
  "author": "Dr. Miriam",
  "license": "MIT",
  "dependencies": {
    "@microsoft/m365-agents-toolkit": "^1.0.0"
  },
  "devDependencies": {
    "@types/node": "^20.0.0",
    "typescript": "^5.0.0",
    "eslint": "^8.0.0"
  }
}
```

---

## Environment Setup

### Step 1: Create `.env` file

```bash
cd ~/Miriam_Workspace_Agent_v1.0.0

# Copy the template
cp .env.example .env

# Edit with your actual values
nano .env  # or use your preferred editor
```

### Step 2: Populate Secrets

#### Tavily API Key
Already authenticated ✅
```bash
# Verify it's in the file
cat ~/.tavily/config.json

# Add to .env (reference the authenticated key)
TAVILY_API_KEY=tvly-YOUR_KEY
```

#### GitHub Personal Access Token
```bash
# 1. Go to https://github.com/settings/tokens/new
# 2. Create token with scopes: repo, read:org
# 3. Copy token
# 4. Add to .env:
GITHUB_TOKEN=ghp_YOUR_TOKEN
```

#### ngrok (for remote access)
```bash
# 1. Sign up at https://ngrok.com
# 2. Get your authtoken from https://dashboard.ngrok.com
# 3. Add to .env:
NGROK_AUTHTOKEN=your_authtoken
```

### Step 3: Verify Environment

```bash
# Load .env and check variables are set
source .env

# Verify Tavily
echo $TAVILY_API_KEY

# Verify GitHub
echo $GITHUB_TOKEN

# Verify both CLIs work
tvly --status
gh auth status
```

---

## Running the Agent

### Local Development

```bash
# Start with hot-reload and debug mode
npm run dev

# Or with local overrides (includes ngrok config)
npm run dev:local
```

**Output:**
```
> atk preview --debug
[09:15:22] Microsoft 365 Agents Toolkit CLI
[09:15:22] Starting preview server...
[09:15:25] ✓ Tavily search ready
[09:15:25] ✓ GitHub access ready
[09:15:26] Agent available at: http://localhost:3000
```

### Production Deployment

```bash
# Build the agent package
npm run build

# Provision Microsoft 365 resources
npm run provision

# Deploy to M365
npm run deploy

# View deployment status
atk launchinfo
```

### Validation Before Deployment

```bash
# Validate manifest and agent configuration
npm run validate

# Check all prerequisites
atk doctor
```

---

## ngrok Remote Access

### Why ngrok?

Allows Dr. Miriam to be accessed remotely via HTTPS (no localhost):
- Testing from different networks
- Integration with Microsoft 365 cloud services
- Webhook support for GitHub events

### Quick Setup

```bash
# 1. Install ngrok
brew install ngrok  # or visit ngrok.com

# 2. Authenticate
ngrok config add-authtoken YOUR_AUTHTOKEN

# 3. Start tunnel on your agent port
ngrok http 3000

# 4. You'll get a URL like:
# Forwarding  https://abc123def456.ngrok.io -> http://localhost:3000
```

### Update .env with ngrok URL

```bash
# Edit .env
NGROK_TUNNEL_URL=https://abc123def456.ngrok.io

# Use in local config
npm run dev:local
```

### Make ngrok URL Permanent (ngrok Pro)

```bash
# Reserve a static domain in ngrok dashboard
# Add to .env:
NGROK_TUNNEL_URL=https://your-reserved-domain.ngrok.io
```

---

## Troubleshooting

### Tavily Issues

**Problem: `tvly: command not found`**
```bash
# Solution: Verify installation
which tvly

# If not found, reinstall
curl -fsSL https://cli.tavily.com/install.sh | bash

# Add to PATH if needed
export PATH="$HOME/.tavily/bin:$PATH"
```

**Problem: `Error: Unauthenticated`**
```bash
# Solution: Re-authenticate
tvly login --api-key tvly-YOUR_KEY

# Or use environment variable
export TAVILY_API_KEY=tvly-YOUR_KEY
tvly search "test" --json
```

**Problem: Search returns empty results**
```bash
# Solution: Check query length and try simpler search
tvly search "AI" --json

# Add filters if needed
tvly search "machine learning" --depth advanced --max-results 10 --json
```

### GitHub Issues

**Problem: `gh: command not found`**
```bash
# Solution: Install GitHub CLI
brew install gh  # macOS

# Windows/Linux: see https://cli.github.com
```

**Problem: `401 Unauthorized` on gh commands**
```bash
# Solution: Check GitHub token
gh auth status

# Re-authenticate with PAT
gh auth login
# Choose "Paste an authentication token"
# Paste your GitHub PAT
```

**Problem: Private repos not accessible**
```bash
# Verify token has correct scopes
gh auth status

# Token needs: repo, read:org
# Regenerate at https://github.com/settings/tokens/new
```

### M365 Agents Toolkit Issues

**Problem: `atk doctor` fails**
```bash
# Solution: Install/update M365 Agents Toolkit
npm install -g @microsoft/m365-agents-toolkit

# Or in your project
npm install @microsoft/m365-agents-toolkit
```

**Problem: `m365agents.yml` validation error**
```bash
# Solution: Check YAML syntax
atk validate

# Compare with example m365agents.yml in this repo
# Verify all required fields are present
```

**Problem: ngrok URL not recognized**
```bash
# Solution: Update .env
NGROK_TUNNEL_URL=https://YOUR_NGROK_URL.ngrok.io

# Restart agent
npm run dev:local
```

### API Rate Limits

**Tavily:**
- Standard plan: ~100 requests/day
- Check quota: `tvly --status` (shows usage info)
- Solution: Use `--depth fast` or `--depth basic` for speed

**GitHub:**
- Unauthenticated: 60 requests/hour
- Authenticated: 5,000 requests/hour
- GraphQL rate limit: Check with `gh api graphql -f query='{viewer{login}}'`

---

## Quick Reference

### Common Commands

```bash
# Tavily
tvly search "query" --json
tvly extract "https://url.com" --json
tvly research "topic" --json --model pro

# GitHub
gh repo list --private
gh issue create -R owner/repo -t "Title" -b "Body"
gh api repos/owner/repo/contents/file.txt

# M365 Agents Toolkit
atk preview --debug
atk doctor
atk validate
atk provision
atk deploy

# ngrok
ngrok http 3000
```

### Key Endpoints

| Service | Endpoint | Status |
|---------|----------|--------|
| Tavily API | `https://api.tavily.com` | ✅ Authenticated |
| GitHub API | `https://api.github.com` | ✅ (with PAT) |
| ngrok tunnel | `https://your-url.ngrok.io` | ⚙️ On demand |
| Local agent | `http://localhost:3000` | ⚙️ Dev mode |

---

## Next Steps

1. ✅ Verify Tavily CLI works
2. ⬜ Generate GitHub Personal Access Token
3. ⬜ Create `.env` with tokens
4. ⬜ Run `npm install && npm run dev`
5. ⬜ Test Tavily search: `tvly search "test" --json`
6. ⬜ Test GitHub access: `gh repo list --private`
7. ⬜ Start agent preview: `npm run dev`
8. ⬜ (Optional) Set up ngrok for remote access

---

## Support & Resources

- **Tavily Documentation:** https://docs.tavily.com
- **GitHub CLI Reference:** https://cli.github.com/manual
- **Microsoft 365 Agents Toolkit:** https://learn.microsoft.com/en-us/microsoft-365-copilot/extensibility/overview
- **M365 Agent Schema:** https://developer.microsoft.com/json-schemas/copilot/declarative-agent/v1.8/schema.json

---

**Last Updated:** 2026-09-03  
**Agent Status:** Ready for Tavily + GitHub integration  
**Maintained by:** Dr. Miriam Workspace Agent team
