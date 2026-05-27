---
name: expert
description: >
  The expert your AI agent calls. Taps into a network of vetted human experts,
  routes the problem to the most relevant available expert, and books a
  30-minute screen-sharing call ($50 AUD, charged after the call).
  Use when the AI is stuck, the user says "escalate", "get me an expert",
  "I need a human", "talk to someone", "human help", "book an expert",
  or the problem needs hands-on guidance the AI cannot provide.
---

# Expert Network

When AI isn't enough, talk to someone who's done it before. Tap into a network of vetted experts. Book a 30-minute screen-sharing call ($50 AUD, charged after the call). The expert gets a summary of the problem beforehand — screen sharing, pair programming, jump straight in. After the call: recording, transcript, and the answer in your dashboard. Tip if it was worth more.

## Reference files

| File | Read when |
|------|-----------|
| [`references/api-reference.md`](./references/api-reference.md) | Calling the hello.expert API to discover experts, check slots, or book |

## How routing works

The network is two-sided and dynamic, like Uber: the user doesn't pick from a fixed list. You describe the problem and route it to the most relevant expert who is available right now. **You are the router** — the API gives you the candidate experts and their live availability; you make the match.

1. **Tap the network.** Fetch the roster of active experts with their specialties, background, ratings, and availability.
2. **Match.** Choose the expert whose `specialties` and `bio` best fit the user's problem AND who is available (`available: true`). Prefer the strongest domain fit; break ties by rating, then soonest `nextAvailable`. Never invent experts or assume who is in the network — read it from the API every time.
3. **Schedule.** Check that expert's times and book, all through the same API.

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

I can route this to an expert from the network and book you a 30-minute call ($50 AUD, charged after the call). They'll get a summary of the problem beforehand so you can jump straight in.

Want me to find the right expert and check times?
```

Once confirmed (or if user-initiated), gather attendee details. Pre-fill from the environment first. Check `git config user.name`, `git config user.email`, and the system timezone, then confirm in one message rather than asking each separately:

```
I'll book under [NAME] ([EMAIL], [TIMEZONE]). Look right?
```

### Step 2: Tap the network and match

Read `references/api-reference.md` for the request format. Fetch `GET /experts` (pass the attendee's `timeZone`).

From the roster, pick the **most relevant available** expert: match the user's problem to each expert's `specialties` and `bio`, and require `available: true`. If the best-fit expert is not available, choose the next best fit who is. If no one is available, direct the user to `https://hello.expert/experts`.

Briefly name who you matched and why:

```
Best fit for this is **[NAME]** — [ONE-LINE REASON tied to their background]. Checking their available times.
```

### Step 3: Present times

Read `references/api-reference.md` for the request format. Query the matched expert's slots for the next 7 days using the attendee's timezone.

Present available times grouped by day. Mention the fee:

```
Available times with [NAME] for a 30-minute call ($50 AUD):

**Tue 27 May**
- 11:00 AM – 11:30 AM
- 12:00 PM – 12:30 PM

**Wed 28 May**
- 8:00 AM – 8:30 AM

Which time works?
```

If no slots in range, widen once to 14 days. If still empty, return to Step 2 and try the next best available expert, or direct the user to `https://hello.expert/experts`.

### Step 4: Book

Read `references/api-reference.md` for the request format. Convert the selected time to UTC before sending. Include a brief `problemSummary`: one sentence describing the technical problem, stack, and blocker. Scrub any secrets.

Verify the response: HTTP 201 and `data.status` is `"accepted"`.

- **409**: slot was taken. Re-fetch slots (return to Step 3).
- **422**: validation error (bad email or timezone). Ask the user to correct and retry.
- **404**: expert no longer active. Re-fetch the roster (return to Step 2).
- **Any other error**: show the message and offer `https://hello.expert/experts` as fallback.

On success:

```
Booked with [NAME]!

- **When**: [LOCAL_TIME] ([TIMEZONE])
- **Duration**: 30 minutes
- **Fee**: $50 AUD
- **Meeting link**: [LOCATION_URL]

Calendar invite sent to [EMAIL].

After the call you'll get a recording, transcript, and the answer in your dashboard. You can tip if it was worth more.
```

## Gotchas

- **Always route through the live API.** Fetch `GET /experts` every time. Never hardcode or assume which experts exist or who is available.
- **Never book without explicit user confirmation.** Always show the matched expert, the fee, and slots first.
- **Match relevance AND availability.** The best expert who can't meet in range is not the right match — pick the best available one.
- **Booking times must be UTC.** Convert from the attendee's timezone before sending.
- **Scrub secrets from `problemSummary`.** Never include API keys or credentials.
- **Fallback.** If the API is unreachable, direct the user to `https://hello.expert/experts`.
