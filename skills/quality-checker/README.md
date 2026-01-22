# Quality Checker Skill

Enforces quality gates during and after task execution.

## Purpose
Verifies that completed work meets project standards before marking tasks complete or merging to main.

## Quality Gates

### Code Quality Gates
- **TypeScript strict mode** - No implicit `any` unless documented
- **Linting passes** - 0 errors (warnings allowed)
- **Unit tests pass** - 100% of tests green
- **Build succeeds** - No compilation errors
- **Coverage maintained** - Don't decrease coverage %

### E2E Gates (for UI changes)
- **Critical flows work** - Main user paths pass
- **No regressions** - Existing tests still pass
- **Accessibility OK** - WCAG AA compliance

### Agent-Native Gates (if applicable)
- **PARITY_MAP updated** - New tools documented
- **CRUD complete** - Agent can create/read/update/delete
- **Completion signals** - Tools return `{ success, shouldContinue }`

### Architecture Gates
- **No circular dependencies** - Code dependency graph is clean
- **Logging present** - Errors are logged, not silent
- **Configuration externalized** - No hardcoded values

## Execution Flow

```
Task Implementation
       ↓
Quality Check Triggered
       ↓
Run All Gates for Project
       ↓
┌─────────────────────┐
│ All Gates Pass?     │
└─────────────────────┘
    ✓ YES → Mark Complete ✓
    ✗ NO  → Show Failures & Retry ↻
```

## Retry Strategy
- Max 3 attempts per gate
- 10 second backoff between retries
- Show full error logs on final failure
- Flag for human review if repeatedly fails

## Example

### Input
```json
{
  "project": "milo",
  "task_id": "quick-1",
  "changes": "committed 13 files",
  "type": "maintenance"
}
```

### Gates Run (from quality-gates.json)
```json
[
  { "name": "git_status_clean", "status": "PASS" },
  { "name": "lint", "status": "PASS" },
  { "name": "build", "status": "FAIL", "error": "TypeScript error in src/index.ts:45" }
]
```

### Output
```json
{
  "task_id": "quick-1",
  "overall_status": "FAILED",
  "gates_passed": 2,
  "gates_failed": 1,
  "failures": [
    {
      "gate": "build",
      "error": "TypeScript error in src/index.ts:45: TS7015: ...",
      "command": "npm run build",
      "action": "FIX_ERROR_AND_RETRY"
    }
  ],
  "next_action": "Fix errors and retry with: dev-execute quick-1 --retry"
}
```

## Usage

### During Task Execution
```bash
quality-checker.sh --project milo --gate typescript,lint,build
```

### After Task Completion
```bash
quality-checker.sh --project milo --all --strict
```

### With Retry
```bash
quality-checker.sh --project milo --all --max-retries 3
```

## Configuration
Gates are defined in `~/Development/config/quality-gates.json` per project.

## Success Metrics
- 95%+ gates pass on first run
- Average retry count < 1.2
- Zero silent failures (all errors surface)
- Quality improves over time (track gate pass rates monthly)
