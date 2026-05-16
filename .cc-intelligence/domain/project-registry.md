---
name: Project Registry
description: This skill should be used when Claude needs to understand, query, or modify the projects registry, detect AI/ML projects, track project metadata, or provide project status information.
version: 1.0.0
---

# Project Registry Skill

## Overview

This skill provides the knowledge and procedures for managing the Central Agent's project registry system, with special focus on AI/ML development projects.

## Registry Location

**Primary Registry**: `%HOME%\Projects\data\projects-registry.json`

## Project Type Classification

### AI/ML Project Types

| Type | Description | Key Indicators |
|------|-------------|----------------|
| `ml-training` | Machine learning model training | models/, training scripts, datasets, .pt/.h5 files |
| `llm-app` | LLM-powered application | langchain, openai/anthropic SDK, prompts/, chains/ |
| `data-pipeline` | Data processing/ETL | ETL scripts, data/, transformers, pandas heavy |
| `rag-system` | Retrieval Augmented Generation | vector stores, embeddings, document loaders |
| `fine-tuning` | Model fine-tuning project | training configs, base model refs, LoRA/QLoRA |
| `agent-system` | AI agent/multi-agent | agents/, tools/, orchestration code |

### General Project Types

| Type | Description | Key Indicators |
|------|-------------|----------------|
| `web-app` | Web application | frontend/, backend/, API routes |
| `automation` | Automation scripts | scripts/, scheduled tasks, cron |
| `library` | Reusable library/package | setup.py, pyproject.toml with [build-system] |
| `api-service` | API/microservice | endpoints/, OpenAPI spec, Dockerfile |

## AI Stack Detection

### Python Dependencies (requirements.txt / pyproject.toml)

| Package | Indicates |
|---------|-----------|
| `anthropic`, `claude-*` | Claude API integration |
| `openai` | OpenAI API integration |
| `google-generativeai` | Gemini API integration |
| `langchain`, `langchain-*` | LangChain framework |
| `llama-index` | LlamaIndex RAG framework |
| `transformers` | Hugging Face models |
| `torch`, `pytorch` | PyTorch deep learning |
| `tensorflow`, `keras` | TensorFlow deep learning |
| `sentence-transformers` | Embedding models |
| `chromadb`, `pinecone`, `weaviate` | Vector databases |
| `ollama` | Local LLM inference |
| `vllm`, `text-generation-inference` | LLM serving |

### JavaScript/TypeScript Dependencies (package.json)

| Package | Indicates |
|---------|-----------|
| `@anthropic-ai/sdk` | Claude API |
| `openai` | OpenAI API |
| `@google/generative-ai` | Gemini API |
| `langchain` | LangChain JS |
| `@pinecone-database/pinecone` | Pinecone vector DB |

### Configuration Files

| File | What to Check |
|------|---------------|
| `.env` | API keys: ANTHROPIC_API_KEY, OPENAI_API_KEY, GOOGLE_API_KEY |
| `config.yaml` | model endpoints, API URLs, provider configs |
| `.claude/CLAUDE.md` | Project-specific Claude instructions |
| `docker-compose.yml` | AI service containers (ollama, vllm, etc.) |

## Project Status Workflow

### Status Values

| Status | Description | Criteria |
|--------|-------------|----------|
| `active` | Currently being developed | Activity in last 7 days |
| `paused` | Temporarily on hold | No activity 7-30 days, not archived |
| `completed` | Finished, maintained | Marked complete, occasional updates |
| `archived` | No longer maintained | No activity 30+ days, marked archived |

### Priority Levels

| Priority | Description | Typical Characteristics |
|----------|-------------|-------------------------|
| `high` | Critical/urgent | Client deliverables, production systems |
| `medium` | Important | Active development, planned features |
| `low` | Background/experimental | Learning projects, experiments |

## Registry Operations

### Query Projects

```python
# By status
active_projects = [p for p in registry["projects"] if p["status"] == "active"]

# By type
ai_projects = [p for p in registry["projects"] if p["type"] in ["ml-training", "llm-app", "rag-system"]]

# By AI stack
claude_projects = [p for p in registry["projects"] if "claude" in p.get("ai_stack", [])]

# By priority
high_priority = [p for p in registry["projects"] if p["priority"] == "high"]

# Stale projects (no activity in 7+ days)
from datetime import datetime, timedelta
cutoff = datetime.now() - timedelta(days=7)
stale = [p for p in registry["projects"]
         if datetime.fromisoformat(p["last_activity"]) < cutoff]
```

### Update Project

```python
def update_project(registry, project_id, updates):
    for project in registry["projects"]:
        if project["id"] == project_id:
            project.update(updates)
            project["last_modified"] = datetime.now().isoformat()
            break
    return registry
```

### Add New Project

```python
import uuid
from datetime import datetime

def add_project(registry, project_data):
    new_project = {
        "id": str(uuid.uuid4()),
        "created": datetime.now().isoformat(),
        "last_activity": datetime.now().isoformat(),
        "status": "active",
        "priority": "medium",
        **project_data
    }
    registry["projects"].append(new_project)
    return registry
```

## Project Health Indicators

### Healthy Project Signs
- Regular commits (at least weekly for active)
- README.md exists and is updated
- Dependencies are pinned/locked
- Tests exist and pass
- .gitignore is comprehensive
- No secrets in code

### Warning Signs
- No activity in 14+ days (for active projects)
- Missing README
- Uncommitted changes for 7+ days
- No .gitignore
- Hardcoded API keys
- No tests

### Critical Issues
- Exposed secrets in git history
- Broken dependencies
- No recent backups
- Missing license for public repos

## Integration with File System

### Project Root Detection

A directory is likely a project root if it contains:
1. `.git/` directory
2. `requirements.txt` OR `package.json` OR `pyproject.toml`
3. `src/` OR `lib/` directory
4. `README.md`

### File Count Categories

| Category | Extensions |
|----------|------------|
| Source Code | .py, .js, .ts, .tsx, .jsx |
| Data | .csv, .json, .parquet, .xlsx |
| Models | .pt, .pth, .h5, .keras, .onnx, .pkl |
| Notebooks | .ipynb |
| Config | .yaml, .yml, .toml, .ini, .env |
| Docs | .md, .rst, .txt |

## Reporting Templates

### Summary Card
```
┌─────────────────────────────────────────┐
│ PROJECT: {name}                         │
│ Type: {type} | Priority: {priority}     │
│ Status: {status}                        │
├─────────────────────────────────────────┤
│ AI Stack: {ai_stack}                    │
│ Last Activity: {relative_time}          │
│ Files: {file_count} | Size: {size_mb}MB │
└─────────────────────────────────────────┘
```

### Health Report
```
Project Health: {name}
═══════════════════════

✅ README exists
✅ Git repository initialized
⚠️  No tests found
❌ API keys in .env not in .gitignore
✅ Dependencies locked (requirements.txt)
⚠️  Last commit: 10 days ago

Health Score: 65/100
Recommendations:
1. Add .env to .gitignore
2. Create tests/ directory
3. Commit recent changes
```

## Cross-Agent Communication

### Notify File Monitor
When a new project is detected, create notification:
```json
{
  "type": "new_project_detected",
  "project_path": "C:\\path\\to\\project",
  "timestamp": "ISO8601"
}
```

### Request Gemini Analysis
For deep project analysis, prepare handoff:
```json
{
  "task_type": "project_audit",
  "payload": {
    "project": "project-name",
    "project_path": "C:\\path",
    "analysis_requested": [
      "architecture_review",
      "dependency_audit",
      "code_quality",
      "security_scan"
    ]
  }
}
```
