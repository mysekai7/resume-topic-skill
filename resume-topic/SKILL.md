---
name: resume-topic
description: Resume a previous conversation topic or session. Triggered by messages like "/resume <keyword>". Searches memory files first, then session JSONL logs, reconstructs context, and continues the conversation.
---

# resume-topic

Restore context from a previous session and continue the conversation.

## Trigger phrases

- `/resume <keyword>`
- `resume <keyword>`
- `continue the previous topic about <keyword>`
- `pick up the last conversation about <keyword>`

## Steps

### 1) Search memory first (fast path)

```bash
rg -n "<keyword>" ~/.openclaw/workspace/memory/ ~/.openclaw/workspace/MEMORY.md 2>/dev/null | head -n 60
```

If memory matches, use it directly and skip the session-log search.

### 2) Search session logs (fallback)

```bash
# Find sessions containing the keyword
rg -l "<keyword>" ~/.openclaw/agents/main/sessions/*.jsonl 2>/dev/null

# Show time range for each matching session
for f in <matched files>; do
  first=$(head -1 "$f" | jq -r '.timestamp')
  last=$(tail -1 "$f" | jq -r '.timestamp')
  size=$(ls -lh "$f" | awk '{print $5}')
  echo "$first -> $last  $size  $(basename $f)"
done | sort -r
```

### 3) Extract relevant content from the best matching session

```bash
# User messages (filtered, no metadata noise)
jq -r 'select(.type=="message" and .message.role=="user") | (.message.content[]? | select(.type=="text") | .text)' <session>.jsonl \
| rg -v 'Conversation info|untrusted metadata|```json|\{|\}|sender_id|message_id|sender:|timestamp:|group_subject|is_group_chat|conversation_label' \
| sed '/^\s*$/d' | head -n 80

# Assistant messages (key outputs)
jq -r 'select(.type=="message" and .message.role=="assistant") | (.message.content[]? | select(.type=="text") | .text)' <session>.jsonl \
| head -n 200
```

### 4) Reconstruct and present context

Summarize in 3–5 bullet points:
- What the user was trying to do
- Key decisions or findings
- Where things were left off / what is pending

Then ask: **"Where should we continue?"** (or continue immediately if the intent is clear).

### 5) Optional: write back to memory

If the topic is important and not captured yet, append a short summary to `~/.openclaw/workspace/memory/YYYY-MM-DD.md`.

## Tips

- If multiple sessions match, present a shortlist and let the user pick
- Large sessions (>1MB): use `head`/`tail` sampling instead of reading the whole file
