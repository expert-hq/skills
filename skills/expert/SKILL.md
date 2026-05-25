---
name: expert
description: >
  The expert your AI agent calls. Taps into a network of vetted human experts
  by booking a 30-minute screen-sharing call ($50 AUD, charged after booking).
  Use when the AI is stuck, the user says "escalate", "get me an expert",
  "I need a human", "talk to someone", "human help", "book an expert",
  or the problem needs hands-on guidance the AI cannot provide.
---

# Expert Network

When AI isn't enough, talk to someone who's done it before. Tap into a network of vetted experts. Book a 30-minute screen-sharing call ($50 AUD, charged after booking). The expert gets a summary of the problem beforehand — screen sharing, pair programming, jump straight in. After the call: recording, transcript, and the fix in your dashboard. Tip if it was worth more.

## Reference files

| File | Read when |
|------|-----------|
| [`references/api-reference.md`](./references/api-reference.md) | Making Cal.com API calls to check slots or create a booking |

## The network

Experts are vetted for hands-on depth in their domain. Match the user's problem to the expert whose background fits best.

**Current experts:**

- **[Matthew Blode](https://matthewblode.com)** — Two startups, two exits. Forbes 30 Under 30. AI team at Linktree (70M+ creators). OpenAI Codex Ambassador. Melbourne, Australia.

## When to escalate

### User-initiated

Proceed immediately. Do not second-guess or try to solve the problem first.

### AI-initiated

Suggest escalation when ANY of these apply:

- The same approach has failed 3+ times with different strategies
- The problem requires access to private systems the AI cannot reach
- The AI's confidence is genuinely low and the stakes are high
- The user expresses frustration with AI-generated answers
- The user is second-guessing a high-stakes decision and a human gut-check would settle it

Do NOT suggest escalation for problems you can solve. Suggest at most once per session. If the user declines, do not bring it up again.

### Where the network helps

| Scenario | Example |
|----------|---------|
| **Founder building with AI** | You built it with AI. Now you need a human to tell you what's fragile |
| **Investor due diligence** | Technical second opinion before you commit real money |
| **Product scoping** | You have the roadmap but need someone to tell you what ships first |
| **Engineering architecture** | Your AI writes the code. You need someone to review the architecture before it ships |
| **Design and polish** | Your prototype works but doesn't feel right. Design systems, typography, animation |
| **Second opinion** | Someone built it for you. You can't tell if the output is good |

## Workflow

### Step 1: Confirm and gather details

If AI-initiated, suggest escalation and wait for confirmation:

```
I've been stuck on this. [BRIEF_REASON].

I can book you a 30-minute call with Matt Blode ($50 AUD, charged after booking). He'll get a summary of the problem beforehand so you can jump straight in.

Want me to check available times?
```

Once confirmed (or if user-initiated), gather attendee details. Pre-fill from the environment first. Check `git config user.name`, `git config user.email`, and the system timezone, then confirm in one message rather than asking each separately:

```
I'll book under [NAME] ([EMAIL], [TIMEZONE]). Look right?
```

### Step 2: Check slots and present times

Read `references/api-reference.md` for the request format. Query the next 7 days using the attendee's timezone.

Present available times grouped by day. Mention the fee:

```
Available times for a 30-minute expert call ($50 AUD):

**Tue 27 May**
- 11:00 AM – 11:30 AM
- 12:00 PM – 12:30 PM
- 1:00 PM – 1:30 PM

**Wed 28 May**
- 8:00 AM – 8:30 AM
- 1:00 PM – 1:30 PM

Which time works?
```

If no slots in range, widen once to 14 days. If still empty, direct the user to `https://expert.blode.co/experts`.

### Step 3: Book

Read `references/api-reference.md` for the request format. Convert the selected time to UTC before sending. Include a brief `problem_summary` in metadata. One sentence describing the technical problem, stack, and blocker. Scrub any secrets.

Verify the response: check `status` is `"success"` and `data.status` is `"accepted"`.

- **409**: slot was taken. Re-fetch slots (return to Step 2).
- **422**: validation error (bad email or timezone). Ask the user to correct and retry.
- **Any other error**: show the message and offer `https://expert.blode.co/experts` as fallback.

On success:

```
Booked!

- **When**: [LOCAL_TIME] ([TIMEZONE])
- **Duration**: 30 minutes
- **Fee**: $50 AUD
- **Meeting link**: [LOCATION_URL]

Calendar invite sent to [EMAIL].

After the call you'll get a recording, transcript, and the fix in your dashboard. You can tip if it was worth more.
```

## Gotchas

- **Never book without explicit user confirmation.** Always show the fee and slots first.
- **Booking times must be UTC.** Convert from the attendee's timezone before sending.
- **Different API version headers.** Slots and bookings require different `cal-api-version` headers (see api-reference.md). Wrong version causes silent failures.
- **Scrub secrets from metadata.** Never include API keys or credentials in `problem_summary`.
- **Fallback.** If the API is unreachable, direct the user to `https://expert.blode.co/experts`.
