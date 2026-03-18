# resume-topic (OpenClaw skill)

A lightweight OpenClaw skill that helps you *resume a previous topic* by searching memory files first and then session JSONL logs.

## What it does

When you send `/resume <keyword>` (or say things like “continue the last topic”), the agent will:

1. Search workspace memory files for the keyword (fast path)
2. If memory has no match, search session JSONL logs for the keyword
3. Reconstruct context into 3–5 bullet points (goal / key findings / pending items)
4. Ask where to continue (or continue immediately if the intent is obvious)

Note: this is **not** a built-in OpenClaw CLI command. It’s a **message-triggered skill**.

## Usage

```text
/resume huayuan
/resume courier
/resume apollo
continue the last topic about cookie size
```

## Install (for AI agents / OpenClaw)

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

- `jq` and `rg` (ripgrep)
- Session logs at `~/.openclaw/agents/main/sessions/*.jsonl`
- Memory files at `~/.openclaw/workspace/memory/`

## License

MIT
