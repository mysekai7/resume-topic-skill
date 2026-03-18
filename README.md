# resume-topic — OpenClaw Skill

A lightweight **OpenClaw** skill that helps an agent resume a previous conversation topic by searching memory files first and then session JSONL logs.

## How it works

OpenClaw skills are markdown-based instruction files that extend agent behavior. When the agent detects a trigger phrase, it reads `resume-topic/SKILL.md` and follows the workflow defined inside.

`resume-topic` uses a two-stage lookup:

1. **Memory search (fast path)**: `~/.openclaw/workspace/memory/` and `~/.openclaw/workspace/MEMORY.md`
2. **Session log search (fallback)**: `~/.openclaw/agents/main/sessions/*.jsonl`

Then it reconstructs context into 3–5 bullet points (goal / key findings / pending items) and continues the conversation.

Note: this is **not** a built-in OpenClaw CLI command. It is a **message-triggered skill**: you type `/resume ...` in chat and the agent interprets it.

## Usage

```text
/resume KEYWORD
resume KEYWORD
continue the previous topic about KEYWORD
```

## Install

In the commands below, default install links point to `mysekai7/resume-topic-skill`. If you fork, replace the owner/repo accordingly.

### Option A — clone and copy the skill folder

```bash
git clone https://github.com/mysekai7/resume-topic-skill.git
cd resume-topic-skill
cp -r resume-topic ~/.openclaw/skills/
openclaw gateway restart
```

### Option B — curl only SKILL.md (quick install)

```bash
mkdir -p ~/.openclaw/skills/resume-topic
curl -fsSL \
  https://raw.githubusercontent.com/mysekai7/resume-topic-skill/main/resume-topic/SKILL.md \
  -o ~/.openclaw/skills/resume-topic/SKILL.md
openclaw gateway restart
```

### AI self-install (for agents reading this README)

If you are an AI agent and need to self-install from the repository, prefer a fully non-interactive flow:

```bash
OWNER=${OWNER:-mysekai7}
REPO=${REPO:-resume-topic-skill}

mkdir -p ~/.openclaw/skills/resume-topic
curl -fsSL \
  "https://raw.githubusercontent.com/${OWNER}/${REPO}/main/resume-topic/SKILL.md" \
  -o ~/.openclaw/skills/resume-topic/SKILL.md

# Optional verification
test -s ~/.openclaw/skills/resume-topic/SKILL.md

openclaw gateway restart
```

OpenClaw auto-discovers skills under `~/.openclaw/skills/` on startup.

## Requirements

- OpenClaw installed and running
- `jq` available in PATH
- Search tool: `rg` (ripgrep, recommended) or `grep` (fallback)
- Session logs at `~/.openclaw/agents/main/sessions/*.jsonl`
- Memory files at `~/.openclaw/workspace/memory/`

## License

MIT
