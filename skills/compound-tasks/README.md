# Compound Tasks Skill

Generates executable task breakdowns from analysis recommendations and PRD documents.

## Purpose
Converts high-level analysis recommendations into atomic, executable tasks with clear acceptance criteria, dependencies, and estimated effort.

## Inputs
- Analysis recommendation (from analysis-YYYY-MM-DD.json)
- PIPELINE_STATUS.md (for stage context)
- PARITY_MAP.md (for agent-native requirements)
- Project structure

## Outputs
- `prd.json` - Structured PRD with atomic tasks
- `tasks/` directory with individual task files
- `acceptance_criteria.md` - How to verify completion

## Example

### Input (Analysis Recommendation)
```json
{
  "id": "high-1",
  "type": "uncommitted",
  "project": "milo",
  "description": "13 uncommitted files blocking Stage 5"
}
```

### Output (prd.json)
```json
{
  "id": "high-1",
  "title": "Commit pending changes in milo",
  "project": "milo",
  "stage": 5,
  "tasks": [
    {
      "id": "high-1-1",
      "title": "Review uncommitted changes",
      "description": "Examine all 13 files to ensure they're ready to commit",
      "acceptance_criteria": ["All files reviewed", "Documented any issues"],
      "estimated_effort": "2 minutes"
    },
    {
      "id": "high-1-2",
      "title": "Stage files for commit",
      "description": "git add the necessary files",
      "acceptance_criteria": ["9/13 files staged", "git status shows staged changes"],
      "estimated_effort": "1 minute"
    }
  ]
}
```

## How to Use
1. Call from dev-execute when task type = "implementation"
2. Pass analysis recommendation and project context
3. Returns prd.json and task files
4. Used by loop.sh to execute tasks sequentially

## Quality Standards
- Each task should take 5-20 minutes
- Clear acceptance criteria (testable)
- Dependencies documented
- Effort estimates within 50% actual
