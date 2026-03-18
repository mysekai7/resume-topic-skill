# resume-topic — OpenClaw Skill

A lightweight **OpenClaw** skill that helps an agent resume a previous conversation topic.

It searches **workspace memory first**, then falls back to **session JSONL logs**, and reconstructs the context into a short actionable summary.

## How it works

OpenClaw skills are markdown-based instruction files that extend agent behavior. When the agent detects a trigger phrase, it reads `resume-topic/SKILL.md` and follows the workflow defined inside.

`resume-topic` uses a two-stage lookup:

1. **Memory search (fast path)**: `~/.openclaw/workspace/memory/` and `~/.openclaw/workspace/MEMORY.md`
2. **Session log search (fallback)**: `~/.openclaw/agents/main/sessions/*.jsonl*` (includes reset logs)

Then it reconstructs context into 3–6 bullet points (goal / what was tried / decisions / current status / next actions).

**Cross-platform support**: Works on macOS, Linux, and Windows (Git Bash/WSL).

## Usage

```text
/resume KEYWORD
resume KEYWORD
继续 KEYWORD
恢复 KEYWORD
接着聊 KEYWORD
```

**Important for Telegram users:**
- Do NOT register `/resume_topic` as a bot command in BotFather
- Telegram commands cannot accept parameters after clicking
- Users must type `/resume <keyword>` directly in chat

## Install

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

## Requirements

- OpenClaw installed and running
- `jq` available in PATH
- Search tool: `rg` (ripgrep, recommended) or `grep` (fallback)
- Session logs under `~/.openclaw/agents/main/sessions/`
- Workspace memory under `~/.openclaw/workspace/memory/`

## License

MIT
