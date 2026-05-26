# Expert Skills

Skill bundle for routing a problem to a vetted human expert and booking a call through the hello.expert API. One skill (`expert`), one reference file (`references/api-reference.md`).

## Structure

```
skills/expert/
  SKILL.md                        - Routing logic, escalation, workflow, gotchas
  references/
    api-reference.md              - hello.expert API (experts, slots, bookings)
```

## How it works

The skill is the router in a two-sided network. It fetches the live roster of active experts from `GET https://hello.expert/api/experts`, matches the user's problem to the most relevant available expert (agent-side, no fixed list), then checks slots and books — all through hello.expert. The backend (github.com/mblode/expert-web) proxies Cal.com; the skill never calls Cal.com directly.

## Constraints

- SKILL.md must stay under 500 lines (agent-skills format spec)
- API details (curl commands, request/response schemas, error codes) belong in `references/api-reference.md`, not in SKILL.md
- Never commit secrets. No API key is needed — the hello.expert endpoints are public
- The `$50 AUD` fee and `hello.expert/experts` URL are the two values that appear in multiple places. Update all occurrences if they change
- Never hardcode expert names in the skill. The roster is always read from the API

## Validation

```bash
# Verify line counts (SKILL.md < 500, api-reference.md < 200)
wc -l skills/expert/SKILL.md skills/expert/references/api-reference.md

# Test the routing endpoint is live
curl -s "https://hello.expert/api/experts?timeZone=Australia/Melbourne" | python3 -m json.tool

# Install and verify skill appears
cp -R skills/expert ~/.claude/skills/expert
```
