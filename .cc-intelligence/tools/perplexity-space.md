# Skill: Perplexity Space Interaction (v1.0 SAt)

**Namespace**: `tools/perplexity-space`
**Objective**: Project-scoped research with tiered context retrieval.

## Interaction Priority
1. **Internal Files**: Query "Files" section for `projects-registry.json` and session logs.
2. **Docs Mirror**: Reference `https://github.com/thebigalex2494/central_command_docs_sync`.
3. **Global Web**: Execute search for delta/updates.

## Operational Constraints
- **Language**: 100% Technical English.
- **Context**: Project-scoped (one project per thread).
- **Tooling**: Browser Scripting (Playwright) or MCP only. No API keys.
- **Reporting**: Short, actionable fragments.

## Conflict Resolution
- **Space vs Mirror**: Space files take precedence.
- **Mirror vs Web**: Mirror takes precedence for architectural rules.
- **Discrepancy**: Mark as [Conflict].

## Execution Flow
1. **Intake**: Identify target Project ID (e.g., `proj-001`).
2. **Scan**: Verify if `projects-registry.json` is available in Space.
3. **Research**: Execute query focusing on project-specific constraints.
4. **Aggregate**: Produce English summary for agent consumption.
