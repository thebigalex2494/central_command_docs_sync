# Skill: BrowserAgent MCP

This skill grants Gemini CLI the ability to perform advanced web navigation, interact with applications (AppSheet), and extract data from external sites (Perplexity, DeepSeek) using a specialized agent with session persistence.

## Identity

- **Name**: `browser-agent-mcp`
- **Engine**: Playwright MCP + Anthropic Claude (Reasoning)
- **Status**: EXPERIMENTAL (Tier 4)
- **Capability**: Google Auth Persistence via `pw-pool`

## Operating Instructions

### 1. Mode Selection
- **Automated Mode (Headless)**: Use for routine extraction tasks where an active session already exists.
- **Login/Maintenance Mode (Headed)**: Mandatory when manual login or captcha solving is required.

### 2. Session Management (pw-pool)
The skill automatically uses the Playwright pool.
- If no free instances are available, it waits or asks the user to run `bash tools/pw-pool/pw-lock.sh cleanup`.
- Upon completion, the lock is automatically released.

## Available Commands

### Standard Execution (Hidden)
```bash
python %CC%\tools\browser_agent_playwright_mcp.py "<objective>"
```

### Maintenance Execution (Visible for Login)
To force visible mode, pass the internal parameter to the tool or use this CLI command:
```bash
# Note: Requires the script to detect intent or use the headless=False parameter if called via MCP
python %CC%\tools\browser_agent_playwright_mcp.py "Navigate to <url> in visible mode for login"
```

## Required Configuration
- `ANTHROPIC_API_KEY`: Key for the reasoning agent.
- `BROWSER_AGENT_ALLOW_AI_DOMAINS`: Must be `true` for testing with Perplexity/DeepSeek.

## Security Rules
1. **No Loops**: The agent has a limit of 10 iterations per task.
2. **Restricted Domain**: Default blocking of AI domains to avoid infinite recursion, controlled bypass via environment variable.
3. **Privacy**: Sessions are saved locally in `tools/pw-pool/sessions/`.
