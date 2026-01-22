# Automated Discovery → Ship Pipeline

A comprehensive system that transforms development workflow from reactive (bug fixes, scattered work) to proactive (intelligent recommendations, guided execution).

## Architecture Overview

```
Phase 0: Nightly (3 AM)
├─ collect-product-health.sh
└─ Outputs: reports/nightly-YYYY-MM-DD.json

Phase 1: Morning Analysis (6 AM)
├─ analyze-report.sh (Claude API)
└─ Outputs: reports/analysis-YYYY-MM-DD.json

Phase 2: Morning Report (7 AM)
├─ generate-morning-report.sh
└─ Outputs: reports/morning-YYYY-MM-DD.md
└─ Opens in editor (📍 HUMAN REVIEW POINT)

Phase 3: User Execution (On-Demand)
├─ dev-execute <task-id>
├─ Quality gates enforced
└─ Outputs: Completed work / Branch / PR

Phase 4: Feedback Tracking
├─ feedback-tracker.sh
└─ Learns patterns for future analysis
```

## Quick Start

### 1. Enable Automation

```bash
bash ~/Development/scripts/setup-automation.sh --enable
```

This installs launchd agents that will run daily:
- **3:00 AM** - Collects product health data
- **6:00 AM** - Analyzes with Claude (requires API key)
- **7:00 AM** - Generates morning report and opens

### 2. Manual Execution (for testing)

```bash
# Collect health data
~/Development/scripts/collect-product-health.sh

# Analyze (requires CLAUDE_API_KEY set)
export CLAUDE_API_KEY="sk-..."
~/Development/scripts/analyze-report.sh

# Generate morning report
~/Development/scripts/generate-morning-report.sh
```

### 3. Execute a Task

```bash
# See available tasks
dev-execute

# Execute a specific task
dev-execute quick-1
dev-execute high-2
```

## How It Works

### Phase 0: Nightly Collection (3 AM)

Gathers comprehensive health metrics:

```json
{
  "git": {
    "uncommitted": [{"project": "milo", "files": 13}],
    "unpushed": [],
    "stale_branches": [{"project": "deepstack", "days_inactive": 31}],
    "summary": {"uncommitted": 2, "unpushed": 0, "stale": 5}
  },
  "pipeline": {
    "products": [...],
    "blockers": [...]
  },
  "agent_parity": {
    "gaps": [],
    "crud_audit": [...]
  },
  "disk_usage": {"total_dev": "21G", "total_id8": "16G"}
}
```

**Data Sources:**
- Git status across all products
- PIPELINE_STATUS.md (current stage, next checkpoint)
- PARITY_MAP.md (agent tool completeness)
- Supabase logs (errors, performance)
- Test results (CI/CD runs)

### Phase 1: Morning Analysis (6 AM)

Claude API analyzes nightly report:

```json
{
  "summary": "...",
  "critical_issues": [],
  "high_priority": [
    {
      "id": "high-1",
      "type": "uncommitted",
      "project": "milo",
      "description": "13 uncommitted files",
      "recommendation": "Commit or stash before Stage 5"
    }
  ],
  "quick_wins": [...],
  "decision_gates": [...],
  "recommendations": {
    "immediate": [...],
    "this_week": [...],
    "next_week": [...]
  }
}
```

### Phase 2: Morning Report (7 AM)

Beautiful, actionable markdown report:

```markdown
# Morning Development Report
**Date:** Thursday, January 22, 2026

## Executive Summary
Portfolio is stable with 5 stale projects...

## Major Decisions (Need Your Input)
_No major decisions needed today_

## Executable Tasks

### Quick Wins (Low Risk, High Impact)
**quick-1**: Commit pending changes in milo
  → Run: `dev-execute quick-1`
  → Effort: < 10 minutes | Risk: low

## How to Proceed
1. Review decisions above
2. Run tasks: `dev-execute <task-id>`
3. System logs outcomes automatically
```

📍 **STOPS HERE** - User reviews and makes decisions

### Phase 3: User-Triggered Execution

User runs: `dev-execute quick-1`

For **simple tasks** (maintenance, cleanup):
- Execute directly
- Run quality gates
- Commit results
- Log feedback

For **complex tasks** (features):
- Generate PRD with atomic tasks
- Create feature branch
- Run execution loop
- Create PR for review

### Phase 4: Quality Gates

All work passes these gates before merging:

| Gate | Purpose |
|------|---------|
| `typescript_strict` | No implicit `any` |
| `lint` | ESLint 0 errors |
| `unit_tests` | All tests pass |
| `build_succeeds` | Compilation works |
| `e2e_tests` | Critical flows work |
| `parity_updated` | Agent tools documented |

If a gate fails, task is retried (max 3 attempts) or flagged for human review.

### Phase 5: Feedback Loop

Each task execution is logged:

```json
{
  "task_id": "quick-1",
  "timestamp": "2026-01-22T10:30:00Z",
  "outcome": "success",
  "duration_minutes": 8,
  "notes": "Committed all 13 files"
}
```

Monthly analysis identifies:
- High-success categories (auto-execute ready)
- High-failure categories (need more rigor)
- Complexity patterns (estimate improvements)
- Pipeline progression (which stages stall)

## File Structure

```
~/Development/
├── scripts/
│   ├── collect-product-health.sh      # Phase 0: Collect metrics
│   ├── analyze-report.sh              # Phase 1: Claude analysis
│   ├── generate-morning-report.sh     # Phase 2: Beautiful report
│   ├── dev-execute                    # Phase 3: Execute tasks
│   ├── feedback-tracker.sh            # Phase 5: Log outcomes
│   └── setup-automation.sh            # Install launchd agents
│
├── config/
│   ├── automation.json                # Global settings
│   ├── quality-gates.json             # Per-project quality rules
│   ├── com.eddieb.dev-health-*.plist # launchd configuration
│   └── feedback-db.json               # Learning database
│
├── reports/
│   ├── nightly-YYYY-MM-DD.json        # Raw health data
│   ├── analysis-YYYY-MM-DD.json       # LLM analysis
│   ├── morning-YYYY-MM-DD.md          # User report
│   └── feedback-db.json               # Learning patterns
│
└── id8/skills-registry/skills/stackshack/
    ├── compound-tasks/                # Generate atomic tasks
    ├── scope-validator/               # Validate scope fence
    └── quality-checker/               # Enforce quality gates
```

## Quality Gate Configuration

Edit `~/Development/config/quality-gates.json` to define per-project requirements:

```json
{
  "per_project": {
    "milo": {
      "quality_gates": ["typescript_strict", "lint", "unit_tests", "build_succeeds"],
      "min_coverage": 75
    },
    "deepstack": {
      "quality_gates": ["typescript_strict", "lint", "build_succeeds"],
      "min_coverage": 70
    }
  }
}
```

## Automation Settings

Edit `~/Development/config/automation.json`:

```json
{
  "enabled": false,
  "schedule": {
    "nightly_collection": {"time": "03:00"},
    "morning_analysis": {"time": "06:00"},
    "morning_report": {"time": "07:00"}
  },
  "automation": {
    "auto_execute_enabled": false,
    "quality_gates_enabled": true,
    "auto_commit": true,
    "auto_create_prs": true
  }
}
```

## Decision Gates

These require explicit human approval:

| Type | Examples | Decision |
|------|----------|----------|
| **Stage Progression** | Advance from Stage 1 → 2 | "Does this meet checkpoint?" |
| **Scope Changes** | Violates "Not Yet" list | "Should we add this now?" |
| **Data Migration** | Schema changes to production | "Approved? Any rollback plan?" |
| **External Integration** | Add new API/service | "Cost/benefit? Alternative?" |
| **Agent Features** | New agent tools | "Tested with agents? Full CRUD?" |

## Troubleshooting

### No morning report appears

Check logs:
```bash
tail ~/Development/reports/launchd-*.log
```

Common issues:
- **3 AM collection fails** - Check git repo permissions
- **6 AM analysis fails** - Ensure `CLAUDE_API_KEY` is set
- **7 AM report fails** - Check if analysis file was created

### Tasks not executing

Verify dev-execute is working:
```bash
dev-execute
# Should show: Available tasks from today's report
```

### Quality gates failing

Check gate definition:
```json
cat ~/Development/config/quality-gates.json | jq '.per_project.milo'
```

Run gates manually:
```bash
cd ~/Development/id8/products/milo
npm run lint
npm test
npm run build
```

## Advanced Usage

### Manual Run All Phases

```bash
# Run collection
~/Development/scripts/collect-product-health.sh

# Run analysis
export CLAUDE_API_KEY="sk-..."
~/Development/scripts/analyze-report.sh

# Generate report
~/Development/scripts/generate-morning-report.sh
```

### Check Automation Status

```bash
bash ~/Development/scripts/setup-automation.sh --status
```

### View Feedback Patterns

```bash
cat ~/Development/reports/feedback-db.json | jq '.patterns'
```

### Generate Report for Specific Date

```bash
# Use existing data
DATE=2026-01-20
jq . ~/Development/reports/nightly-$DATE.json | \
  /Users/eddiebelaval/Development/scripts/analyze-report.sh
```

## Integration with ID8 Pipeline

This automation respects PIPELINE_STATUS.md:

- **Stage-Appropriate**: Doesn't recommend Stage 5 work if in Stage 1-3
- **Scope Fence**: Won't recommend "Not Yet" features
- **Agent-Native**: Ensures PARITY_MAP.md is kept current
- **Quality Gates**: Enforces before Stage advancement

### Updating PIPELINE_STATUS.md

When a task reaches checkpoint, update manually:

```markdown
## Current Stage: 5 - Feature Blocks
## Checkpoint: "Does this feature work completely?"

### Blockers
- [ ] All tests passing
- [ ] TypeScript types strict
- [ ] Agent CRUD complete

### Not Yet
- Twilio integration (Q2 2026)
- Email notifications
- Advanced analytics

### Progress
- [x] Feature A: Complete
- [x] Feature B: Complete
- [ ] Feature C: In progress
```

## Success Metrics

Track these monthly:

1. **Coverage** - % of potential insights surface in reports
2. **Precision** - % of recommended tasks are valuable
3. **Execution Rate** - % of recommendations actually executed
4. **Quality** - % of tasks pass quality gates on first try
5. **Learning** - Month-over-month improvement in all above

## Next Steps

1. **Enable automation**: `bash ~/Development/scripts/setup-automation.sh --enable`
2. **Wait for 7 AM** or run manually for testing
3. **Review morning report** - open and read at 7 AM
4. **Execute a task** - `dev-execute quick-1`
5. **Track progress** - System logs outcomes automatically
6. **Review monthly patterns** - Check feedback-db.json on 1st of month

---

_Last Updated: 2026-01-22_
_Pipeline Status: Phase 2 Complete (Reporting Working) | Phase 3-5 In Development_
