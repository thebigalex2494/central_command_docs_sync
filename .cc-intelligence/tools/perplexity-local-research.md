# SKILL: Perplexity Local Research (Zero-Token Path)
## Namespace: tools/perplexity-local-research

### Description
Allows the agent to perform deep web research using Perplexity's browser interface (bypassing API) and analyze the results using a local Ollama model. This ensures maximum privacy and zero cloud token expenditure for the research/analysis phase.

### Prerequisites
- Python 3.10+
- Playwright installed and browsers focused (`playwright install chromium`)
- Ollama running locally with `qwen2.5-coder:latest` or similar.

### Execution Protocol
To invoke this skill, the agent must execute the integrated research script via the terminal.

**Command (Invisible):**
```powershell
python C:\Users\msi\central_command\scripts\perplexity_module\integrated_research.py "YOUR_QUERY_HERE"
```

**Command (Visible - Recommended if Cloudflare appears):**
```powershell
python C:\Users\msi\central_command\scripts\perplexity_module\integrated_research.py "YOUR_QUERY_HERE" --visible
```

**Optional Model Override:**
```powershell
python C:\Users\msi\central_command\scripts\perplexity_module\integrated_research.py "YOUR_QUERY_HERE" --model "deepseek-coder-v2:16b"
```

### Constraints
1. **Intelligence Plane**: The local model performs the analysis. Gemini only orchestrates the call and presents the final summary.
2. **Speed**: Expect 45-90 seconds for full execution (Browser startup + Ollama inference).
3. **Selector Stability**: If the script fails with a TimeoutError, check the `#ask-input` selector in `perplexity_browser_scraper.py`.

### Output Handling
- Capture the `STDOUT` from the command.
- The script will output a section labeled `FINAL INTEGRATED SUMMARY (Local Analysis)`.
- Present this summary to the user as the authoritative local-first research result.

---
**Status**: `READY` | **Protocol**: `WSL/Win Hybrid` | **Plane**: `Intelligence`
