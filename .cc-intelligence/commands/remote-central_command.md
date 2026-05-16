# 🚀 Command: /remote-central_command
> **Goal**: Absolute session migration. Packs current chat context and prepares for immediate remote resume.

## 👤 Agent Action
When the user invokes this command, Antigravity MUST:
1. **Summarize**: Generate a high-density summary of the entire conversation (Objective, Decisions, Blockers, Code changes).
2. **Export**: Run `python %CC%/tools/handoff.py` using this summary.
3. **Persist**: Save the detailed summary to `.cc-intelligence/sessions/BRIDGE_{project_id}.md`.
4. **Handoff**: Provide the user with the final instruction to close the current IDE session.

## 📥 Remote Resume
The agent on the other side (Gemini CLI) will:
1. Load the JSON.
2. Read the `BRIDGE_{project_id}.md` to regain "consciousness" of the full chat history.
3. Confirm: "Full Context Restored. Ready to continue."
