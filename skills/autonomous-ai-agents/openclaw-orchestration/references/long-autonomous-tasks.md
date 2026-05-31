# Long Autonomous Tasks — When and How

## When `openclaw agent --message` is NOT enough

Use case: "Build a PWA", "Implement a feature", "Refactor this codebase"
→ These require 30–80+ tool calls over minutes/hours.
→ `openclaw agent --message` sends ONE message and gets ONE reply. The agent STOPS.
→ Mission Control Board Orgchart will show **orange** (idle) immediately after.

## The Right Pattern: Claude Code as Background Process

```bash
# 1. Prepare a task file
mkdir -p ~/my-build && cat > ~/my-build/TASK.md << 'EOF'
[Full task description here]
EOF

# 2. Launch Claude Code as a background process
source ~/.hermes/.env
terminal(background=true, notify_on_complete=true):
  source ~/.hermes/.env && cd ~/my-build && \
  ANTHROPIC_API_KEY=$ANTHROPIC_API_KEY claude -p "$(cat TASK.md)" \
    --dangerously-skip-permissions \
    --max-turns 80 \
    --output-format json > /tmp/build-result.json 2>/tmp/build-log.txt
  echo "EXIT: $?"

# 3. Verify it started
sleep 20 && ps aux | grep "claude" | grep -v grep
ls -la ~/my-build/

# 4. Read result when notified
cat /tmp/build-result.json | python3 -c "
import json,sys; d=json.load(sys.stdin)
print('Status:', d.get('subtype'))
print('Turns:', d.get('num_turns'))
print('Cost:', d.get('total_cost_usd'))
print('Result:', d.get('result','')[:2000])
"
```

## Verification Checklist

Before telling the user "the task is running":
- [ ] `ps aux | grep claude` shows the process
- [ ] Session/build directory has files appearing
- [ ] Mission Control Board shows green OR cost counter is ticking

## luna-voice Build Context (May 2026)

- Repo: `titan-aven/luna-voice` (GitHub, public)
- Local: `~/luna-voice-build/`
- Stack: FastAPI WebSocket server + PWA client
- Port: 8765 (8443 was taken by Tailscale)
- TLS: `tailscale cert mac-mini-von-aven.tail15c773.ts.net`
- LaunchAgent: `~/Library/LaunchAgents/com.aven.luna-voice.plist`
- OpenAI Key: from `~/.openclaw/openclaw.json` → `messages.tts.providers.openai.apiKey`
- Hermes API: `http://127.0.0.1:8642/v1`
- iPhone install URL: `https://mac-mini-von-aven.tail15c773.ts.net:8765`

## Why Not Zoe for Build Tasks?

Zoe (Paperclip CEO, Opus model) is for strategy and issue decomposition.
Build tasks with clear requirements → go directly to Claude Code or Titan.
Routing through Zoe adds latency + cost with no benefit when requirements are already defined.

## Root Cause: Why Titan's Session Died (May 2026 Post-Mortem)

Titan actually built the whole luna-voice project correctly (08:25–08:30). 
The session was killed by `"This operation was aborted"` error in `openclaw:prompt-error`.

**Cause:** My `openclaw agent --message` call timed out after 60s on Luna's side.
This severed the Gateway HTTP connection → Gateway aborted Titan's running session mid-step.

**Evidence:** `~/.openclaw/agents/titan/sessions/e343a08b...jsonl` shows Titan working
through all steps, then abrupt abort at 08:30:07 with `"This operation was aborted"`.

**Fix Applied:** `BUILD_JOBS.md` placed in `~/.openclaw/workspace-titan/` instructing Titan
to always: (1) confirm receipt in < 5s, (2) detach build via `nohup ... &`, (3) post result to Discord when done.

**Luna-side fix:** Always use `--timeout 600` when sending long-running tasks to agents:
```bash
openclaw agent --agent titan --thinking off --timeout 600 \
  --session-key "agent:titan:discord:channel:1487563493526732860" \
  --message "..." --deliver --reply-channel discord \
  --reply-to "channel:1487563493526732860" --reply-account titan
```

## Delegating to Titan: The Correct Pattern

```
Luna → openclaw agent (--timeout 600, --thinking off) →
  Titan receives → immediately replies "starting background job" →
  Titan runs: nohup claude -p "$(cat TASK.md)" --dangerously-skip-permissions ... &
  Titan posts result to #titan channel when done
```

NOT:
```
Luna → openclaw agent → Titan builds synchronously → timeout kills session mid-build
```
