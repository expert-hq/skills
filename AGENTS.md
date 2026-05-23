# Expert Skills

Skill bundle for booking human expert calls via Cal.com. One skill (`expert`), one reference file (`references/api-reference.md`).

## Structure

```
skills/expert/
  SKILL.md                        — Escalation logic, workflow, gotchas
  references/
    api-reference.md              — Cal.com v2 slots + bookings API
```

## Constraints

- SKILL.md must stay under 500 lines (agent-skills format spec)
- API details (curl commands, request/response schemas, error codes) belong in `references/api-reference.md`, not in SKILL.md
- Never commit secrets — the Cal.com API key is not needed (slots and bookings are public endpoints)
- The `$50 AUD` fee and `cal.com/mblode/expert` URL are the two values that appear in multiple places — update all occurrences if they change

## Validation

```bash
# Verify line counts (SKILL.md < 500, api-reference.md < 200)
wc -l skills/expert/SKILL.md skills/expert/references/api-reference.md

# Test Cal.com slots endpoint is live
curl -s "https://api.cal.com/v2/slots?eventTypeSlug=expert&username=mblode&start=$(date +%Y-%m-%d)&end=$(date -v+5d +%Y-%m-%d)&timeZone=Australia/Melbourne&format=range" -H "cal-api-version: 2024-09-04" | python3 -m json.tool

# Install and verify skill appears
cp -R skills/expert ~/.claude/skills/expert
```
