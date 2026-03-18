---
name: resume-topic
description: Resume a previous conversation topic or session. Use when the user says things like "/resume", "继续上次的话题", "恢复 xxx 话题", "上次我们聊到...", "找找 xxx 相关的对话", or wants to pick up where a previous session left off. Searches session logs and memory files to reconstruct context, then continues the conversation naturally.
---

# resume-topic

Restore context from a previous session and continue the conversation.

## Trigger phrases
- `/resume <keyword>`
- "继续上次的 xxx 话题"
- "恢复 xxx 的讨论"
- "上次聊到 xxx，继续"
- "找找 xxx 相关的 session"

## Steps

### 1. Search memory files first (fast)
```bash
rg -n "<keyword>" ~/.openclaw/workspace/memory/ ~/.openclaw/workspace/MEMORY.md 2>/dev/null | head -n 60
```
Memory files are pre-summarized — if the topic is there, use it directly and skip step 2.

### 2. Search session logs if memory has no match
```bash
# Find sessions containing the keyword
rg -l "<keyword>" ~/.openclaw/agents/main/sessions/*.jsonl 2>/dev/null

# Get time range for each matching session
for f in <matched files>; do
  first=$(head -1 "$f" | jq -r '.timestamp')
  last=$(tail -1 "$f" | jq -r '.timestamp')
  size=$(ls -lh "$f" | awk '{print $5}')
  echo "$first -> $last  $size  $(basename $f)"
done | sort -r
```

### 3. Extract relevant content from the best matching session
```bash
# User messages (filtered, no metadata noise)
jq -r 'select(.type=="message" and .message.role=="user") | (.message.content[]? | select(.type=="text") | .text)' <session>.jsonl \
| rg -v 'Conversation info|untrusted metadata|```json|\{|\}|sender_id|message_id|sender:|timestamp:|group_subject|is_group_chat|conversation_label' \
| sed '/^\s*$/d' | head -n 80

# Assistant key outputs
jq -r 'select(.type=="message" and .message.role=="assistant") | (.message.content[]? | select(.type=="text") | .text)' <session>.jsonl \
| head -n 200
```

### 4. Reconstruct and present context
Summarize what was discussed in 3-5 bullet points:
- What the user was trying to do
- Key decisions or findings
- Where things were left off / what was pending

Then ask: "要从哪里继续？" or directly continue if the intent is clear.

### 5. Optionally update memory
If the topic has useful context not yet in memory, append a summary to `memory/YYYY-MM-DD.md`.

## Tips
- If multiple sessions match, show a list and let the user pick
- CST = UTC+8; convert timestamps when showing times to the user
- Session logs are in `~/.openclaw/agents/main/sessions/*.jsonl`
- Memory files are in `~/.openclaw/workspace/memory/`
- Large sessions (>1MB): use `head`/`tail` sampling instead of reading the whole file
