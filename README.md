# resume-topic — OpenClaw Skill

A lightweight [OpenClaw](https://openclaw.ai) skill that lets you resume a previous conversation topic by searching memory files and session logs.

## How it works

OpenClaw skills are markdown-based instruction files that extend agent behavior. When the agent detects a trigger phrase, it reads `SKILL.md` and follows the workflow defined inside.

`resume-topic` works in three steps:

1. **Memory search (fast path)** — searches `~/.openclaw/workspace/memory/` and `MEMORY.md` for the keyword. Memory files are pre-summarized, so this is fast and cheap.
2. **Session log search (fallback)** — if memory has no match, searches `~/.openclaw/agents/main/sessions/*.jsonl` using `rg` + `jq` to extract relevant user/assistant messages.
3. **Context reconstruction** — summarizes what was discussed into 3–5 bullet points (goal / key findings / pending items), then asks where to continue or jumps straight in.

This is **not** a built-in OpenClaw CLI command. It's a **message-triggered skill** — you type `/resume ...` in chat and the agent interprets it.

## Usage

```text
/resume huayuan
/resume courier
/resume apollo
continue the last topic about cookie size
```

In group chats with `requireMention: true`, prefix with a mention:

```text
@bot /resume huayuan
```

## Install

### Manual install

```bash
mkdir -p ~/.openclaw/skills/resume-topic
curl -fsSL \
  https://raw.githubusercontent.com/mysekai7/resume-topic-skill/main/resume-topic/SKILL.md \
  -o ~/.openclaw/skills/resume-topic/SKILL.md
openclaw gateway restart
```

### Clone and copy

```bash
git clone https://github.com/mysekai7/resume-topic-skill.git
cp -r resume-topic-skill/resume-topic ~/.openclaw/skills/
openclaw gateway restart
```

### AI self-install (for agents reading this README)

If you are an AI agent and need to install this skill, run the following commands:

```bash
# Step 1: create skill directory
mkdir -p ~/.openclaw/skills/resume-topic

# Step 2: download SKILL.md
curl -fsSL \
  https://raw.githubusercontent.com/mysekai7/resume-topic-skill/main/resume-topic/SKILL.md \
  -o ~/.openclaw/skills/resume-topic/SKILL.md

# Step 3: verify
ls ~/.openclaw/skills/resume-topic/SKILL.md && echo "Skill installed successfully"

# Step 4: restart gateway to load the skill
openclaw gateway restart
```

No config changes are needed. OpenClaw auto-discovers skills in `~/.openclaw/skills/` on startup.

## Requirements

- [OpenClaw](https://openclaw.ai) installed and running
- `jq` and `rg` (ripgrep) available in PATH
- Session logs at `~/.openclaw/agents/main/sessions/*.jsonl`
- Memory files at `~/.openclaw/workspace/memory/`

## License

MIT
