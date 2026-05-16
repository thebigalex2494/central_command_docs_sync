# 📡 Skill: Remote Handoff (v1.0 SAt)
> **Goal**: Facilitate seamless session and context migration between Local (Antigravity) and Remote (Gemini CLI) nodes.

## 👤 Role & Purpose
- **Antigravity**: Acts as the *Exporter* (Source). Prepares the cognitive and environmental state.
- **Gemini CLI**: Acts as the *Importer* (Target). Re-hydrates the session state.

---

## 🛠 Commands

### `/handoff [objective] [line_of_thought]`
**Agent**: Antigravity (Local)
1. Executes `python %CC%/tools/handoff.py`.
2. Captures Git status, active project metadata, and environment snapshot.
3. Generates the `SESSION_HANDOFF.json` in `.cc-intelligence/sessions/`.

### `/resume`
**Agent**: Gemini CLI (Remote)
1. Reads `.cc-intelligence/sessions/SESSION_HANDOFF.json`.
2. Validates timestamp and project ID.
3. Injects the `cognitive_bridge` into the active system prompt.
4. Alerts about `dirty_files` to prevent out-of-sync edits.

### `/handoff-clear`
**Agent**: Any
1. Deletes the current handoff file to prevent stale resumes.

---

## 📄 Data Contract (Schema v1.0)
The state is persisted in: `%CC%/.cc-intelligence/sessions/SESSION_HANDOFF.json`

Key fields:
- `cognitive_bridge`: Compressed line of thought (<150 tokens).
- `dirty_files`: List of modified files and their plane (Windows/WSL).
- `sip_phase`: Current SIP stage to resume from.

---

## ⚠️ Safety Protocols
- **Sync First**: Always perform a Git commit or sync before handoff if working on different filesystems.
- **Plane Awareness**: Do not attempt to edit `source_plane: windows` files from a remote Linux node without using the RPC Bridge.
- **Stale State**: Handoffs older than 4 hours should be treated with caution (use `/resume --validate`).
