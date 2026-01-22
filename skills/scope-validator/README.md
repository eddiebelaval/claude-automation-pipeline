# Scope Validator Skill

Validates that analysis recommendations respect project scope boundaries.

## Purpose
Ensures automation doesn't recommend work outside the project's defined "Not Yet" list or Stage progression timeline.

## Validation Rules

1. **Stage Appropriateness**
   - If project is in Stage 1-3, don't recommend Stage 5+ features
   - Recommend only checkpoint-appropriate work

2. **"Not Yet" List Compliance**
   - Check PIPELINE_STATUS.md for "Not Yet" section
   - Flag any recommendations that violate scope fence
   - Example: If "Not Yet: Twilio integration", don't recommend voice calling

3. **Agent-Native Compliance** (if applicable)
   - If product has agent features, ensure PARITY_MAP gaps aren't skipped
   - Don't recommend UI work before agent parity is complete

4. **Data Migration Safety**
   - Flag if recommendation requires schema changes
   - Require human approval for production data changes

## Example

### Input
```json
{
  "id": "high-1",
  "project": "homer",
  "recommendation": "Add Twilio voice calling"
}
```

### Validation Against PIPELINE_STATUS.md
```markdown
## Not Yet
- Twilio integration (Q2 2026)
- Email notifications
- Advanced analytics
```

### Output
```json
{
  "task_id": "high-1",
  "status": "BLOCKED",
  "reason": "scope_fence_violation",
  "message": "Recommendation violates 'Not Yet' list - Twilio integration planned for Q2 2026",
  "action": "FLAG_FOR_HUMAN_REVIEW"
}
```

## Usage
Called before dev-execute when:
- Task type = "feature" or "implementation"
- Recommendation requires architectural decision
- Task touches agent tools or data models

## Pass Criteria
- ✓ Task is appropriate for current stage
- ✓ Task respects "Not Yet" scope fence
- ✓ Task aligns with agent parity goals (if applicable)
- ✓ No production data changes without approval

## Fail Criteria
- ✗ Violates "Not Yet" list → FLAG_FOR_HUMAN_REVIEW
- ✗ Wrong stage for work → REJECT (recommend earlier checkpoint)
- ✗ Production data change → REQUIRE_EXPLICIT_APPROVAL
- ✗ Breaks agent parity → FLAG_FOR_AGENT_AUDIT
