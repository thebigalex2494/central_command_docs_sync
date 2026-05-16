# PLAYWRIGHT AUTOMATION CONTEXT — Central_Command Space
## Technical Reference Document for Local Agent Execution

***

## 1. TARGET ENVIRONMENT

```
Platform      : Perplexity AI (perplexity.ai)
Target URL    : https://www.perplexity.ai/spaces/central-command-5dPMu7AZQce2Huzerr225A
Space Name    : Central_Command
Auth Required : Yes — Active Perplexity Pro session (cookies/session storage)
Tabs          : [Threads] | [Customize]
Query Param   : ?tab=customize  (direct access to Customize tab)
```

***

## 2. MODEL SELECTION — LOCAL AGENT RECOMMENDATION

```
TASK: Perplexity Space Browser Automation via Playwright
RECOMMENDED LOCAL MODELS (ranked by fit):

┌──────────────────────────────────────────────────────────────────────┐
│ TIER 1 — PRIMARY (Best for DOM reasoning + code generation)          │
│  • qwen2.5-coder:32b       — Best-in-class for Playwright scripting  │
│  • deepseek-coder-v2:16b   — Strong selector generation + JS logic   │
│  • codestral:22b           — Fast code completion, low latency       │
├──────────────────────────────────────────────────────────────────────┤
│ TIER 2 — ORCHESTRATION (Best for multi-step planning + reasoning)    │
│  • mistral-nemo:12b        — Efficient multi-step task breakdown      │
│  • llama3.1:8b             — Lightweight, fast for decision trees     │
│  • phi4:14b                — Strong reasoning, low memory footprint   │
├──────────────────────────────────────────────────────────────────────┤
│ TIER 3 — VALIDATION (Best for output verification + error handling)  │
│  • llama3.2:3b             — Ultra-fast response validation           │
│  • gemma2:9b               — Reliable assertion checking             │
└──────────────────────────────────────────────────────────────────────┘

RECOMMENDED PIPELINE:
  qwen2.5-coder:32b  →  Script generation & selector logic
  mistral-nemo:12b   →  Step orchestration & retry strategies
  llama3.2:3b        →  Assertion validation & error classification

CONTEXT WINDOW REQUIREMENT: Minimum 32k tokens
PREFERRED RUNTIME: Ollama local server | LM Studio | llama.cpp
```

***

## 3. FULL PAGE STRUCTURE MAP

### 3.1 Space Header (Both Tabs)
```
Element          : Space title
Current Value    : "Central_Command"
Edit Method      : Direct inline click on title text
Selector Hints   : h1, [class*="space-name"], [class*="title"]
Save Trigger     : Enter key OR click outside element

Element          : Space description
Current Value    : "Describe your project, goals, subject, etc..."
Edit Method      : Direct inline click on description text
Selector Hints   : [class*="description"], p:below(h1)
Save Trigger     : Enter key OR click outside element

Element          : Options button
Selector         : button[aria-label*="more"], button:has-text("...")
Location         : Top-right header bar
Actions Available: Settings, Delete Space, Transfer ownership
```

### 3.2 Tab Navigation
```
Tab 1 — Threads
  Selector      : button:has-text("Threads")
  URL State     : base URL (no query param)

Tab 2 — Customize
  Selector      : button:has-text("Customize")
  URL State     : ?tab=customize
  Direct Nav    : https://www.perplexity.ai/spaces/central-command-5dPMu7AZQce2Huzerr225A?tab=customize
```

***

## 4. THREADS TAB — INTERACTION MAP

### 4.1 Thread List Structure
```
Each thread row contains:
  [TYPE ICON] [THREAD TITLE (truncated)] [USER AVATAR] [TIMESTAMP] [...BUTTON on hover]

Thread Types:
  🖥️  Control browser   — selector: [class*="control-browser"], text="Control browser"
  🔬  Deep research     — selector: [class*="deep-research"],   text="Deep research"
  🔍  Search            — selector: [class*="search"],          text="Search"

Current threads:
  1. [CB]  "Ayudame a generar un prompt para obtener una vista completa de [https://..."
  2. [DR]  "Research status and frameworks of proj-001 (Sales Reports) in this Space c..."
  3. [SR]  "🚀 Perplexity Audit Prompt: Central Command Skill Optimality Role: Lead R..."
  4. [SR]  "Ayudame con lo siguiente, quiero dar de alta un Space llamado Central_Co..."
```

### 4.2 Thread Context Menu (on hover → click "...")
```
TRIGGER:
  1. await threadRow.hover()          // REQUIRED — button only visible on hover
  2. await page.waitForTimeout(300)   // Wait for DOM render
  3. await threadRow.locator('button').last().click()  // Click "..."

MENU OPTIONS:
  ┌─────────────────────────────────────────────────────┐
  │ 📌 Pin             → Pins thread to top of list     │
  │ ✏️  Rename          → Inline rename input activates  │
  │ 🔄 Swap Space      → Move thread to another Space   │
  │ ✖️  Remove from Space → Detach without deleting     │
  │ 🔒 Make private    → Restrict visibility            │
  │ 🗑️  Delete          → Permanent deletion (confirm)  │
  └─────────────────────────────────────────────────────┘

Selector: [role="menuitem"]:has-text("OPTION_NAME")
          [role="menu"] >> text="OPTION_NAME"
```

### 4.3 RENAME Thread — Full Playwright Flow
```javascript
/**
 * RENAME A SINGLE THREAD
 * @param {Page} page - Playwright page object
 * @param {string} textFragment - Partial text to identify the thread
 * @param {string} newName - New name to assign
 */
async function renameThread(page, textFragment, newName) {
  // 1. Locate thread row by partial text
  const threadRow = page.locator('div[class*="thread"], li, [role="listitem"]')
    .filter({ hasText: textFragment })
    .first();

  // 2. Hover to reveal "..." button — CRITICAL STEP
  await threadRow.hover();
  await page.waitForTimeout(300);

  // 3. Click "..." context menu trigger
  await threadRow.locator('button').last().click();

  // 4. Wait for menu to appear
  await page.waitForSelector('[role="menu"]', { timeout: 3000 });

  // 5. Click Rename
  await page.locator('[role="menuitem"]:has-text("Rename")').click();
  await page.waitForTimeout(400);

  // 6. Clear current name and type new one (inline input)
  await page.keyboard.press('Control+A');
  await page.keyboard.type(newName);
  await page.keyboard.press('Enter');

  // 7. Verify rename succeeded
  await page.waitForTimeout(600);
  const confirmed = await page.locator(`text=${newName}`).isVisible();
  console.log(`[RENAME] ${confirmed ? '✅' : '❌'} "${newName}"`);
  return confirmed;
}
```

### 4.4 BATCH RENAME — Naming Convention Strategy
```javascript
/**
 * NAMING CONVENTION:
 * [TYPE] YYYY-MM-DD — Short descriptive title
 *
 * TYPE PREFIXES:
 *   [CB]    = Control Browser automation thread
 *   [DR]    = Deep Research thread
 *   [SR]    = Standard Search thread
 *   [SETUP] = Initial configuration / onboarding
 *   [OPS]   = Internal operations / maintenance
 *   [AUDIT] = Auditing / review threads
 */

const THREAD_RENAME_MAP = [
  {
    fragment: 'Ayudame a generar un prompt para obtener una vista completa',
    newName:  '[CB] 2026-05-15 — Playwright Full Audit Central_Command',
  },
  {
    fragment: 'Research status and frameworks of proj-001',
    newName:  '[DR] 2026-05-15 — Research Status proj-001 Sales Reports',
  },
  {
    fragment: 'Perplexity Audit Prompt: Central Command Skill',
    newName:  '[AUDIT] 2026-05-15 — Skill Optimality Central Command v4.7',
  },
  {
    fragment: 'Ayudame con lo siguiente, quiero dar de alta un Space',
    newName:  '[SETUP] 2026-05-15 — Initial Space Creation Central_Command',
  },
];

async function batchRenameThreads(page) {
  await page.locator('button:has-text("Threads")').click();
  await page.waitForTimeout(800);

  for (const item of THREAD_RENAME_MAP) {
    await renameThread(page, item.fragment, item.newName);
    await page.waitForTimeout(700); // rate-limit buffer
  }
}
```

***

## 5. CUSTOMIZE TAB — INTERACTION MAP

### 5.1 Space Instructions
```
Section Title  : "Space instructions"
Description    : "Give Computer instructions for how it should work in this space."
Current Content: Lead Research Agent role definition (Central Command ecosystem)
Edit Trigger   : Click "Edit instructions" link inside the text box
Input Type     : Textarea (multi-line)
Save Trigger   : Button "Save" or "Done" (appears after edit mode activates)

Playwright:
  await page.locator('text=Edit instructions').click();
  await page.locator('textarea').fill('YOUR NEW INSTRUCTIONS HERE');
  await page.locator('button:has-text("Save")').click();
```

### 5.2 Files
```
Current Files:
  1. proj_001_source_report.md
  2. projects_registry_json.md
  3. GEMINI.md
  4. config_py.md
  + more behind "View all files" link

Actions:
  ADD FILE    : click "+ Add files..."    → system file picker opens
  VIEW ALL    : click "View all files"    → expands full file list
  FILE OPTIONS: click "..." next to file  → rename / delete options
  DELETE ALL  : click 🗑️ icon in section header

Playwright:
  // Add file
  await page.locator('text=Add files').click();
  // Handle file chooser
  const [fileChooser] = await Promise.all([
    page.waitForEvent('filechooser'),
    page.locator('text=Add files').click()
  ]);
  await fileChooser.setFiles('/path/to/your/file.md');

  // View all files
  await page.locator('text=View all files').click();

  // File-level options
  await page.locator('text=proj_001_source_report.md')
    .locator('..').locator('button').last().click();
```

### 5.3 Skills
```
Current Skills:
  1. browser-agent
  2. duckdb-etl-pattern
  3. financial-glossary
  4. project-architect
  + more behind "View all skills" link

Actions:
  ADD SKILL   : click "+ Add skill..."    → skill catalog modal opens
  VIEW ALL    : click "View all skills"   → expands full skill list
  SKILL OPT.  : click "..." next to skill → edit / remove options

Playwright:
  await page.locator('text=Add skill').click();
  await page.locator('[role="option"]:has-text("SKILL_NAME")').click();

  // View all skills
  await page.locator('text=View all skills').click();
```

### 5.4 Links
```
Current Links:
  1. github.com/thebigalex2494/central_command_docs_sync

Actions:
  ADD LINK    : click "+ Add link..."     → URL input appears
  LINK OPT.   : click "..." next to link  → edit / remove options

Playwright:
  await page.locator('text=Add link').click();
  await page.locator('input[placeholder*="URL"], input[type="url"]').fill('https://your-url.com');
  await page.keyboard.press('Enter');
```

***

## 6. CHAT INPUT BAR — SEARCH MODE & MODEL SELECTION

```
Location       : Bottom of the Space page (sticky footer)
Placeholder    : "Start a task in Central_Command"
Context Scope  : "Shared in Space ▾" dropdown (top-left of input box)
```

### 6.1 Switch Between Search and Deep Research
```
Element        : Button "Search ▾" (left side of input bar, with 🔍 icon)
Dropdown Items :
  ✅ Search          (default)
  🔬 Deep research
  🖥️  Control browser

Playwright:
  // Activate Deep Research mode
  await page.locator('button:has-text("Search")').click();
  await page.waitForSelector('[role="option"], [role="menuitem"]');
  await page.locator('[role="option"]:has-text("Deep research")').click();

  // Revert to Search mode
  await page.locator('button:has-text("Deep research")').click();
  await page.locator('[role="option"]:has-text("Search")').click();

  // Activate Control Browser mode
  await page.locator('button:has-text("Search")').click();
  await page.locator('[role="option"]:has-text("Control browser")').click();
```

### 6.2 Change the AI Model
```
Element        : Button "Model ▾" (right side of input bar, before mic icon)
Available Models (per Space instructions):
  • Claude 5.0 Opus (Thinking)  — Complex Orchestration [RECOMMENDED]
  • Claude 4.5 Opus             — Architecture / Planning
  • Claude Code (CLI Agent)     — Implementation / Coding
  • GPT-5.5 (Frontier)          — General Fallback

Playwright:
  // Open model picker
  await page.locator('button:has-text("Model")').click();
  await page.waitForSelector('[role="option"], [role="listbox"]');

  // Select specific model
  await page.locator('[role="option"]:has-text("Claude 5.0 Opus")').click();
  // or
  await page.locator('[role="option"]:has-text("GPT-5.5")').click();
```
