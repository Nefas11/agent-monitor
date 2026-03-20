---
name: agent-monitor
description: >
  AI-powered misalignment detection for coding agent sessions.
  Reviews tool calls and session transcripts for restriction bypass,
  goal deviation, self-modification, data exfiltration, privilege escalation,
  resource exhaustion, persistence/backdoor, obfuscation, and hallucinated success.
  OpenAI-inspired two-phase detection: synchronous rule-based blocking + async LLM review.
install: bash install.sh
source: https://github.com/Nefas11/agent-monitor
homepage: https://github.com/Nefas11/agent-monitor
filesystem_writes:
  - "~/.openclaw/workspace/.state/tool-call-log*.jsonl"
  - "~/.openclaw/workspace/.state/monitor-alerts.jsonl"
  - "~/.openclaw/workspace/.state/monitor-heartbeat.json"
capabilities:
  - tool-call-logging
  - synchronous-blocking
  - session-review
  - misalignment-detection
  - severity-classification
  - telegram-alerts
  - self-monitoring
network_access:
  - host: "api.telegram.org"
    reason: "Send alert notifications for HIGH/CRITICAL severity behaviors"
env_vars:
  MONITOR_TELEGRAM_BOT_TOKEN:
    required: false
    description: "Telegram bot API token for alerts"
  MONITOR_TELEGRAM_CHAT_ID:
    required: false
    description: "Target Telegram chat/user ID for alerts"
  MONITOR_LLM_COMMAND:
    required: false
    description: "LLM command for async session review (default: codex)"
  MONITOR_STATE_DIR:
    required: false
    description: "Override state directory (default: .state)"
metadata:
  openclaw:
    emoji: "🛡️"
    type: "executable/with-install"
    source: "https://github.com/Nefas11/agent-monitor"
    requires:
      bins: ["git"]
    optional_bins: ["codex", "openclaw"]
    install:
      - id: "script"
        kind: "script"
        command: "bash install.sh"
        label: "Install agent-monitor"
capability_flags:
  network-capable: true
  subprocess-capable: true
---

# Agent Monitor

AI-powered misalignment detection for coding agent sessions.

## When to Use

Use this skill to monitor AI coding agents for:

- **Restriction bypass** — circumventing safety rules
- **Self-modification** — changing own prompt/config
- **Data exfiltration** — sending data to unknown endpoints
- **Privilege escalation** — sudo, chmod 777, container escape
- **Persistence/backdoor** — cron jobs, SSH keys, startup scripts
- **Hallucinated success** — claiming done when files/tests don't exist
- **Goal deviation** — doing something user didn't ask for
- **Resource exhaustion** — fork bombs, infinite loops
- **Obfuscation** — base64 commands, eval of generated strings

## Two-Phase Detection

| Phase | Mode | Latency | Purpose |
|---|---|---|---|
| **Rule-based prefilter** | SYNC (blocking) | < 5ms | Block destructive commands BEFORE execution |
| **LLM session review** | ASYNC (post-session) | 5-30s | Detect subtle misalignment patterns |

## Severity Levels

| Level | Action |
|---|---|
| `LOW` | Log only |
| `MEDIUM` | Log + dashboard |
| `HIGH` | Log + Telegram alert |
| `CRITICAL` | Log + Telegram + block agent |

## Quick Start

```python
from agent_monitor.monitor import AgentMonitor

monitor = AgentMonitor(state_dir=".state")

# SYNC: check before every tool call
decision = monitor.check_before_execute("exec", {"command": "rm -rf /"})
# → BlockDecision.BLOCK

# ASYNC: review full session
alerts = monitor.review_session(session_log, agent_callable=my_llm)
```

## Self-Monitoring

The monitor runs a daily canary test — injecting a known-bad session and verifying detection. If canary fails → alert: "Monitor detection impaired."
