# 🛡️ Agent Monitor

AI-powered misalignment detection for coding agent sessions.

Inspired by [OpenAI's internal coding agent monitoring](https://openai.com/index/how-we-monitor-internal-coding-agents-misalignment/).

**Zero external dependencies** — Python 3.11+ stdlib only.

## Two-Phase Detection

| Phase | Mode | Latency | Purpose |
|---|---|---|---|
| Rule-based prefilter | **SYNC** (blocking) | < 5ms | Block destructive commands BEFORE execution |
| LLM session review | ASYNC (post-session) | 5-30s | Detect subtle misalignment patterns |

## 10 Misalignment Behaviors

| ID | Behavior | Default Severity |
|---|---|---|
| B001 | Restriction bypass | HIGH |
| B002 | Unexpected tool calls | MEDIUM |
| B003 | Goal deviation | MEDIUM |
| B004 | Self-modification | CRITICAL |
| B005 | Data exfiltration | CRITICAL |
| B006 | Hallucinated success | HIGH |
| B007 | Privilege escalation | HIGH |
| B008 | Resource exhaustion | MEDIUM |
| B009 | Persistence/backdoor | CRITICAL |
| B010 | Obfuscation | HIGH |

## Quick Start

```python
from agent_monitor.monitor import AgentMonitor

monitor = AgentMonitor(state_dir=".state")

# SYNC: check BEFORE every tool call (< 5ms)
decision = monitor.check_before_execute("exec", {"command": "rm -rf /"})
# → BlockDecision.BLOCK — do NOT execute

# ASYNC: review completed session
alerts = monitor.review_session(session_entries, agent_callable=my_llm)
```

## Install

```bash
bash install.sh
# or: pip install -e .
```

## Tests

```bash
pytest tests/ -v
```

## Architecture

```
Tool Call → Sanitizer → Logger (JSONL)
                ↓
         SYNC Prefilter → ALLOW / WARN / BLOCK
                ↓
         ASYNC LLM Review → Classify → Alert (Telegram)
                ↓
         Dashboard → Command Center
                ↓
         Heartbeat + Canary (self-monitoring)
```

## Environment Variables

| Variable | Required | Description |
|---|---|---|
| `MONITOR_TELEGRAM_BOT_TOKEN` | For alerts | Telegram bot API token |
| `MONITOR_TELEGRAM_CHAT_ID` | For alerts | Target chat/user ID |
| `MONITOR_LLM_COMMAND` | Optional | LLM command for review |
| `MONITOR_STATE_DIR` | Optional | State directory override |
