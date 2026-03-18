---
name: resume-topic
description: Resume a previous conversation topic by searching OpenClaw workspace memory first, then session JSONL logs. Triggered by messages like "/resume <keyword>".
---

# resume-topic

Restore context from a previous session and continue the conversation with minimal back-and-forth.

## Trigger phrases

- `/resume <keyword>`
- `resume <keyword>`
- `继续 <keyword>`
- `恢复 <keyword>`
- `接着聊 <keyword>`
- `pick up the last conversation about <keyword>`

## Workflow

### 0) Confirm keyword

If the user didn’t provide a keyword, ask a single question

- “要从哪个关键词恢复上下文”

### 1) Locate OpenClaw dirs

Use these defaults, but tolerate custom setups via env vars

```bash
WORKSPACE="${OPENCLAW_WORKSPACE:-$HOME/.openclaw/workspace}"
MEM_DIR="$WORKSPACE/memory"
MEM_FILE="$WORKSPACE/MEMORY.md"
SESS_DIR="${OPENCLAW_SESSIONS_DIR:-$HOME/.openclaw/agents/main/sessions}"
KEYWORD="<keyword>"
```

### 2) Search memory first (fast path)

Search `memory/` and `MEMORY.md` first, because they’re curated and cheaper to read

```bash
if command -v rg >/dev/null 2>/dev/null; then
  rg -n -S -F --no-messages -- "$KEYWORD" "$MEM_DIR" "$MEM_FILE" 2>/dev/null | head -n 60
else
  grep -RInF -- "$KEYWORD" "$MEM_DIR" "$MEM_FILE" 2>/dev/null | head -n 60
fi
```

If memory matches are sufficient, summarize them and **skip** session-log search.

### 3) Search session logs (fallback)

Important

- Include reset logs too, they look like `*.jsonl.reset.*`
- Prefer **recent** sessions first
- Don’t print huge blobs
- Guard against whitespace in paths by treating match output as newline-delimited

```bash
# Find matching session files (includes *.jsonl and *.jsonl.reset.*)
if command -v rg >/dev/null 2>/dev/null; then
  MATCHED_FILES=$(rg -l -F --no-messages --glob '*.jsonl*' -- "$KEYWORD" "$SESS_DIR" 2>/dev/null | head -n 40)
else
  MATCHED_FILES=$(grep -RlF --include '*.jsonl*' -- "$KEYWORD" "$SESS_DIR" 2>/dev/null | head -n 40)
fi

echo "$MATCHED_FILES"
```

If nothing matches, say so and ask for a different keyword.

Example: "没找到包含「关键词」的记录，换个词试试？"

#### Show a shortlist with timestamps

```bash
# Print a compact index for the user (most recent by mtime)
echo "$MATCHED_FILES" | while IFS= read -r f; do
  [ -n "$f" ] || continue
  first=$(head -1 "$f" 2>/dev/null | jq -r '.timestamp // empty')
  last=$(tail -1 "$f" 2>/dev/null | jq -r '.timestamp // empty')
  # Cross-platform stat: macOS uses -f, Linux uses -c, Windows fallback to ls
  if stat -f '%Sm' -t '%Y-%m-%d %H:%M:%S' "$f" >/dev/null 2>&1; then
    mtime=$(stat -f '%Sm' -t '%Y-%m-%d %H:%M:%S' "$f")
  elif stat -c '%y' "$f" >/dev/null 2>&1; then
    mtime=$(stat -c '%y' "$f" | cut -d'.' -f1)
  else
    # Fallback for Windows/Git Bash
    mtime=$(ls -l --time-style=long-iso "$f" 2>/dev/null | awk '{print $6" "$7}' || echo "unknown")
  fi
  size=$(ls -lh "$f" | awk '{print $5}')
  echo "$mtime  $size  ${first:-?} -> ${last:-?}  $(basename "$f")"
done | sort -r | head -n 12
```

Pick the best session using this priority

1. Most recent mtime
2. Session where the keyword appears in **user** messages, not only in logs

If multiple look relevant, present the shortlist and let the user choose.

### 4) Extract only the relevant parts from the chosen session

Let `SESSION` be the chosen file path.

#### 4.1 Extract user messages and keep it clean

```bash
jq -r '
  select(.type=="message" and (.message.role=="user"))
  | (.message.content[]? | select(.type=="text") | .text)
' "$SESSION" 2>/dev/null \
| { if command -v rg >/dev/null 2>/dev/null; then
      rg -v --no-messages '^(Conversation info|Sender \(untrusted metadata\)|```json|\{\s*$|\}\s*$|sender_id|message_id|timestamp:|sender:|group_subject|is_group_chat|conversation_label)';
    else
      grep -Ev '^(Conversation info|Sender \(untrusted metadata\)|```json|\{\s*$|\}\s*$|sender_id|message_id|timestamp:|sender:|group_subject|is_group_chat|conversation_label)';
    fi; } \
| sed '/^[[:space:]]*$/d' \
| head -n 120
```

#### 4.2 Show keyword-centered context (fast and high-signal)

```bash
# Show context around the keyword in assistant+user text messages
jq -r '
  select(.type=="message")
  | (.message.role // "") as $r
  | (.message.content[]? | select(.type=="text") | .text) as $t
  | "[" + $r + "] " + $t
' "$SESSION" 2>/dev/null \
| { if command -v rg >/dev/null 2>/dev/null; then
      rg -n -C 2 -F --no-messages -- "$KEYWORD";
    else
      grep -n -C 2 -F -- "$KEYWORD";
    fi; } \
| head -n 120
```

#### 4.3 Extract assistant outputs (optional)

```bash
jq -r '
  select(.type=="message" and (.message.role=="assistant"))
  | (.message.content[]? | select(.type=="text") | .text)
' "$SESSION" 2>/dev/null \
| head -n 200
```

### 5) Reconstruct context for the user

Summarize in 3–6 bullet points

- User’s goal
- What’s been tried / decided
- Key commands / files / links
- Current status
- Next 1–3 actions

Then ask one question

Example: "要继续这个话题吗？还是换个方向？"

- “我们从哪一步继续”

If intent is crystal-clear, continue immediately.

### 6) Optional: write back to memory

If this topic is important and not captured yet, append a short summary to today’s memory file

```bash
TODAY=$(date +%Y-%m-%d)
OUT="$MEM_DIR/$TODAY.md"
mkdir -p "$MEM_DIR"
{
  echo "## Resumed: $KEYWORD"
  echo "- source session: $(basename "$SESSION")"
  echo "- summary: <3-6 bullets>"
  echo
} >> "$OUT"
```

## Notes

- Prefer memory as ground truth, logs are noisy
- For very large sessions, avoid raw dumps, use keyword-centered extraction
- If `jq` is missing, install it or fall back to plain `rg -n -C 2 "$KEYWORD" "$SESSION"`
