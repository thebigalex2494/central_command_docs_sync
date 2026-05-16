# 🧠 Mega-Prompt: Central Command Session Handoff (v1.0 SAt)
> **Usage**: Inject this block into a new session to restore cognitive continuity.

```xml
<SESSION_RESUME_PROTOCOL>
<CONTEXT_SSOT>
Architecture: Central Command v4.7 (Dual-Plane: Windows Host + WSL Intelligence Plane).
Standard: SIP (Audit -> Identify -> Propose -> Execute -> Validate).
Skill_Active: remote-handoff v1.0.
</CONTEXT_SSOT>

<COGNITIVE_BRIDGE>
Objective: {objective}
Line_of_Thought: {line_of_thought}
Next_Action: {next_action}
</COGNITIVE_BRIDGE>

<ENVIRONMENT_SNAPSHOT>
Project_ID: {project_id}
Active_Files: {active_files}
Dirty_Files_Status: {git_status}
Device_Profile: {device_profile}
</ENVIRONMENT_SNAPSHOT>

<AGENT_DIRECTIVES>
1. Preserve Caveman Mode (max token efficiency).
2. Use path: /mnt/c/Users/msi/central_command/ for all filesystem operations.
3. Do not re-audit environment; resume from EXECUTE phase.
4. If encountering "dirty" files, alert user before attempting destructive edits.
</AGENT_DIRECTIVES>
</SESSION_RESUME_PROTOCOL>
```
