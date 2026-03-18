# resume-topic

An OpenClaw skill that restores context from a previous conversation topic/session and continues the chat naturally.

## What it does

When you send `/resume <keyword>` (or say things like “继续上次的话题”), the agent will:

1. Search workspace memory files for the keyword (fast path)
2. If memory has no match, search session JSONL logs for the keyword
3. Reconstruct context into 3–5 bullet points (goal / key findings / pending items)
4. Ask where to continue (or continue immediately if the intent is obvious)

Important: this is **not** an OpenClaw built-in command. It’s a **message-triggered skill**. You type `/resume ...` in chat and the agent interprets it.

## Usage examples

```text
/resume 华苑
/resume courier
继续上次的 apollo 话题
上次聊到 cookie 过长导致 nginx 问题，继续
```

## Install (for AI agents / OpenClaw)

### Option A — clone this repo then copy skill folder

```bash
git clone https://github.com/mysekai7/openclaw-workspace-amsi.git
cd openclaw-workspace-amsi
cp -r skills/resume-topic ~/.openclaw/skills/resume-topic
openclaw gateway restart
```

### Option B — curl only SKILL.md (quick install)

```bash
mkdir -p ~/.openclaw/skills/resume-topic
curl -fsSL \
  https://raw.githubusercontent.com/mysekai7/openclaw-workspace-amsi/main/skills/resume-topic/SKILL.md \
  -o ~/.openclaw/skills/resume-topic/SKILL.md
openclaw gateway restart
```

## Requirements

- `jq` and `rg` (ripgrep)
- Session logs: `~/.openclaw/agents/main/sessions/*.jsonl`
- Memory files: `~/.openclaw/workspace/memory/`

## Notes

If you enable “require mention” in group chats, use `@bot /resume <keyword>`.
