# HYDRA Multi-Agent System Setup

HYDRA (Hybrid Unified Dispatch and Response Architecture) is a multi-agent coordination system that combines Eddie's automation empire with Bhanu's Mission Control patterns.

## Architecture Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                    HYDRA SYSTEM                                  │
├─────────────────────────────────────────────────────────────────┤
│  AGENTS: MILO (coordinator) + FORGE + SCOUT + PULSE             │
│  DATABASE: ~/.hydra/hydra.db (SQLite)                           │
│  SYNC: hydra-sync.sh (8:30 AM daily via launchd)                │
│  INTEGRATION: OpenClaw Gateway (port 18789)                     │
└─────────────────────────────────────────────────────────────────┘
```

## Agent Roster

| Agent | Role | Model | Heartbeat | Cost |
|-------|------|-------|-----------|------|
| MILO | Coordinator | Claude Sonnet 4 | 15 min | ~$10/day |
| FORGE | Dev Specialist | DeepSeek V3.2 | 30 min | FREE |
| SCOUT | Research/Marketing | Qwen3 235B | 60 min | FREE |
| PULSE | Ops Specialist | Llama 4 Maverick | 30 min | FREE |

## Directory Structure

```
~/.hydra/
├── hydra.db              # SQLite coordination database
├── init-db.sql           # Database schema
├── config/
│   └── agents.yaml       # Agent roster configuration
└── sessions/
    ├── milo/             # Coordinator workspace
    │   ├── SOUL.md
    │   ├── IDENTITY.md
    │   ├── AGENTS.md
    │   └── HEARTBEAT.md
    ├── forge/            # Dev specialist workspace
    ├── scout/            # Research specialist workspace
    └── pulse/            # Ops specialist workspace

~/Development/scripts/
└── hydra-sync.sh         # Automation → HYDRA sync

~/Library/LaunchAgents/
└── com.hydra.sync.plist  # 8:30 AM daily schedule
```

## Installation

### 1. Initialize Database

```bash
sqlite3 ~/.hydra/hydra.db < ~/.hydra/init-db.sql
```

### 2. Load Launchd Job

```bash
launchctl load ~/Library/LaunchAgents/com.hydra.sync.plist
```

### 3. Verify Setup

```bash
# Check database
sqlite3 ~/.hydra/hydra.db "SELECT * FROM agents;"

# Check launchd
launchctl list | grep hydra

# Manual sync test
~/Development/scripts/hydra-sync.sh
```

## Usage

### Check Agent Workload

```bash
sqlite3 ~/.hydra/hydra.db "SELECT * FROM v_agent_workload;"
```

### View Pending Tasks

```bash
sqlite3 ~/.hydra/hydra.db "SELECT id, title, assigned_to, priority FROM tasks WHERE status = 'pending';"
```

### View Notifications

```bash
sqlite3 ~/.hydra/hydra.db "SELECT * FROM v_pending_notifications;"
```

## Integration with OpenClaw

HYDRA agent sessions will be registered with the OpenClaw gateway. Each agent workspace (`~/.hydra/sessions/{agent}/`) contains the personality files that OpenClaw loads on session start.

## Related Files

- `scripts/hydra-sync.sh` - Automation sync script (this repo)
- `~/.hydra/` - HYDRA configuration (outside repo)
- `~/.openclaw/` - OpenClaw configuration (outside repo)

## Background

HYDRA is inspired by:
- **Eddie's Automation Empire**: 23 launchd jobs for signal detection
- **Bhanu's Mission Control**: Multi-agent coordination via shared database

The hybrid approach combines automated signal detection with intelligent agent routing.

---

*Created: 2026-02-05*
*Article: "Building an AI-Human Operating System v2" (pending)*
