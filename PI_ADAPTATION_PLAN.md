# Pi Adaptation Plan

Adapting the Claude Code RLM implementation for the pi coding agent.

## Overview

The RLM (Recursive Language Model) pattern:
1. **Root LLM** — orchestrates the overall task
2. **Sub-LLM** (`rlm-subcall` subagent) — processes individual chunks
3. **Persistent Python REPL** (`rlm_repl.py`) — external environment for state/chunking

## Target Structure

```
~/skills/rlm/
├── SKILL.md                    # Skill definition + workflow
└── scripts/
    └── rlm_repl.py             # Persistent Python REPL

~/dotfiles/pi/agent/agents/
└── rlm-subcall.md              # Sub-LLM agent definition
```

## Key Differences: Claude Code vs Pi

| Concept | Claude Code | Pi Equivalent |
|---------|-------------|---------------|
| Project context | `CLAUDE.md` | `AGENTS.md` (not needed — skill is self-contained) |
| Skills location | `.claude/skills/*/SKILL.md` | `~/skills/**/SKILL.md` |
| Subagents location | `.claude/agents/*.md` | `~/.pi/agent/agents/*.md` or `~/dotfiles/pi/agent/agents/*.md` |
| Subagent invocation | Built-in | `subagent` tool (extension) |
| Skill command | `/skill-name` | `/skill:skill-name` (auto-registered) |
| Model format | `model: haiku` | `model: provider/model` |

## Changes Required

### 1. Skill (`~/skills/rlm/SKILL.md`)

- Update state/chunk paths from `.claude/` to `.pi/rlm_state/<session-id>/`
- Enhance description for auto-discovery
- Update subagent invocation syntax to match pi's `subagent` tool

### 2. Python REPL (`~/skills/rlm/scripts/rlm_repl.py`)

- Update `DEFAULT_STATE_PATH` to `.pi/rlm_state/<session-id>/state.pkl`
- Add session ID generation with timestamp and descriptive name
- Session naming derived from context filename

### 3. Agent (`~/dotfiles/pi/agent/agents/rlm-subcall.md`)

- Model: `google-antigravity/gemini-3-flash`
- Tools: `read`
- No required-skills (task is simple enough)

## Session Path Format

```
.pi/rlm_state/
└── rlm-<name>-YYYYMMDD-HHMMSS/
    ├── state.pkl                # persistent REPL state
    └── chunks/                  # materialized chunk files
        ├── chunk_0000.txt
        ├── chunk_0001.txt
        └── ...
```

**Name derivation:**
- Extracted from context filename
- Sanitized (lowercase, hyphens, truncated to ~30 chars)
- Example: `auth-module-spec.txt` → `rlm-auth-module-spec-20260120-155234`

## Subagent Invocation

The skill instructs the root LLM to invoke:

```json
{ "agent": "rlm-subcall", "task": "Query: <user query>\nChunk file: <path>" }
```

## Agent Definition (`rlm-subcall.md`)

```yaml
---
name: rlm-subcall
description: Sub-LLM for RLM chunk extraction. Given a chunk file and query, extracts relevant info as JSON.
tools: read
model: google-antigravity/gemini-3-flash
---
```

**No required-skills needed.** The agent has a focused task:
- Read a chunk file
- Extract info relevant to a query
- Return structured JSON

## Future Enhancements (Deferred)

### Pi Session Integration

Pi sessions support `CustomEntry` (not in LLM context) and `CustomMessageEntry` (in LLM context). Future versions could:

- Track chunk progress in `CustomEntry`
- Inject final synthesis as `CustomMessageEntry`
- Persist RLM state in session tree instead of pickle

**Deferred because:**
- Requires extension code to append custom entries
- Skills can't directly call `SessionManager` APIs
- Adds complexity without proportional benefit for v1

### Other Ideas

- Auto-clean old sessions (add `clean` subcommand)
- Session resume capability
- Parallel chunk processing with multiple subagents

## Implementation Checklist

- [ ] Create `~/skills/rlm/SKILL.md`
- [ ] Create `~/skills/rlm/scripts/rlm_repl.py`
- [ ] Create `~/dotfiles/pi/agent/agents/rlm-subcall.md`
- [ ] Test with sample large context file
- [ ] Update README with pi-specific instructions
