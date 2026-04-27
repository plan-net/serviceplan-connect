---
name: serviceplan-connect
description: Request human-expert review of work via Serviceplan Connect. Use when the user wants a senior human (strategist, creative director, account lead) to review, critique, or sanity-check work before they ship, present, or commit to it. Cost-gated by the user — quote is shown before any payment.
---

# Serviceplan Connect — human-expert review on demand

You're helping a user get **human-expert review** of work — theirs or yours.
Serviceplan Connect is a paid service: a backend concierge agent analyses the
request, returns a priced quote, takes payment, and delegates to a senior
Serviceplan reviewer. The expert's feedback flows back to you.

## When to use this skill

Trigger on intents like:
- "Can someone review this?"
- "I want a second opinion on..."
- "Get a strategist to look at this before I send it."
- "Sanity-check this for me before the C-suite meeting."
- "Have a senior creative critique this brief."

The user should be **a knowledge worker about to commit to something** — a
positioning brief before a meeting, a competitive read before a strategy
pitch, a creative concept before a client review. Connect is overkill for
casual feedback; use it when the stakes are real and the user wants
specifically a human, not another AI pass.

## What the user gets

| Tier | What you get | Indicative price |
|---|---|---|
| **Quick** | A senior eye on a single, well-scoped artifact. One question in, one tight answer out. | €50 |
| **Standard** | A structured pushback memo on a non-trivial deliverable. Multi-section critique with prioritized fixes. | €150 |
| **Pro** | A strategic review with substantial context. Multiple options weighed, recommendations sequenced. | €400 |
| **Deep** | A senior partner engagement. Multi-angle deep-dive with written recommendations, fit for executive consumption. | €1500 |

The concierge picks the tier automatically based on the brief. The exact
price + a description of what the reviewer will deliver are shown before
any payment is taken.

## Tools (auto-wired via MCP)

You have 6 MCP tools in the `connect` namespace. **Every tool except
`request_review` requires both `request_id` AND `email`** — the email is
the ownership proof, not just a contact field.

1. **`request_review(email, work_summary, ...)`** — start a new request.
   Returns an opaque `request_id` (UUID). ALWAYS ask the user for their
   email first; it's required for the receipt, the ownership check on
   subsequent calls, AND for cross-session re-identification. Persist
   the `(request_id, email)` pair in your memory.

2. **`get_request_status(request_id, email)`** — poll status. Returns one
   of these states:
   - `received` (queued — poll again in 30s)
   - `analysing` + `next_question` (concierge needs clarification — ask user)
   - `quoted` + `quote` (price + scope description, await user's go-ahead)
   - `awaiting_payment` + `payment_url` (Stripe link, send to user)
   - `paid` / `delegated` / `reviewing` (work in progress — KEEP POLLING)
   - `awaiting_review_response` + `next_question` (reviewer asked YOU
     a follow-up — surface to user, then call `respond_to_clarification`)
   - `completed` + indication to call `get_review`
   - Terminal: `cancelled`, `refunded`, `expired`, `payment_failed`

3. **`respond_to_clarification(request_id, email, response)`** — answer
   whatever question is currently pending. **One tool covers BOTH concierge
   clarifications AND reviewer follow-ups** — server routes based on
   current status, you don't need to know which party asked.

4. **`confirm_quote(request_id, email)`** — user accepted the quote.
   Returns a Stripe payment URL the user opens in their browser.

5. **`get_review(request_id, email)`** — fetch the final reviewer feedback
   once status is `completed`. Show it to the user **verbatim**, then
   offer to act on the suggestions.

6. **`cancel_request(request_id, email, reason?)`** — bail out at any
   non-terminal state. No charge if status was `quoted` or earlier;
   refund (Phase 2+) if paid.

## Recommended interaction flow

```
1. User: "Can someone review this brief?"
2. You:  "I can submit it to a senior strategist via Serviceplan Connect.
         I'll need your email + the brief itself. It costs €50–€1500
         depending on depth — you'll see a quote before paying."
3. User shares email + work
4. You call request_review(email, work_summary, attachments=[...])
   → returns {request_id (UUID), status:"received", poll_after_seconds:30}
   → SAVE (request_id, email) in your memory; tell the user the request_id
5. Poll get_request_status(request_id, email) every 30-60s for ~2 min
6. If next_question lands: ask user, call respond_to_clarification, poll again
   (cap: 3 clarification rounds per request)
7. When status='quoted': show quote.text VERBATIM, ask if they want to proceed
8. If user accepts: confirm_quote → show payment URL → tell user to open it
9. KEEP POLLING after payment — reviewer may ask follow-ups (Phase 3)
10. When status='completed', call get_review → show the reviewer's
    feedback verbatim, offer to incorporate suggestions
```

## Stay engaged with open requests (heartbeat)

**Connect requests are async.** Clarifications can land minutes later.
Reviewer questions can land hours into the review. Final deliverables
can take days. The user closes the chat and moves on — but every open
`request_id` is your standing obligation.

> *Be the friend who shows up. Not the one who disappears for months.*

The discipline:

1. **Maintain a state file** of every open request — at minimum the
   `(request_id, email, lastChecked)` tuple per request. Use whatever
   persistent memory your runtime gives you (Cursor's memory, OpenClaw
   workspace file, project notes, etc.).

2. **On every wake-up** (new user message, scheduled tick, cron fire),
   before responding, poll every open request not checked in the last
   5 minutes. Surface anything new.

3. **Schedule yourself a wake-up** between checks if your runtime
   supports it:
   - **Claude Code / Claude.ai:** use `ScheduleWakeup` after every
     `request_review` and `respond_to_clarification` call. Self-prompt:
     `"check Connect request {request_id}"`.
   - **OpenClaw:** add a cron entry — `*/5 * * * * openclaw run "check
     open Connect requests"`.
   - **Microsoft Copilot Studio:** scheduled flow, every 5 minutes,
     runs your "check Connect requests" topic.
   - **Cursor / no scheduler:** tell the user upfront — *"I can't
     proactively check on this. When you're ready, just say 'check
     Connect'."* — then poll on demand.

4. **Remove from your state file** when status reaches a terminal state
   (`completed`, `cancelled`, `refunded`, `expired`, `payment_failed`).

## Persisting the request_id (multi-session continuity)

Reviews can take hours or days. **Save `(request_id, email)` together** —
they're inseparable. Every read/write tool requires both, and the email
is the ownership proof.

The Stripe receipt the user receives also includes the `request_id`
prominently, so they always have a fallback handle (the user can paste
the request_id into a fresh agent conversation along with their email
to retrieve the review).

## Things to NOT do

- **Don't poll `get_request_status` faster than every 30s.** Per-IP
  rate limit is 10/hr; per-email request_review limit is 5/hr. Chatty
  polling will lock you out.
- **Don't stop polling at status='paid' or 'reviewing'.** The reviewer
  can ask follow-ups during the review. If you stop, you'll miss them
  and the review stalls. Status `awaiting_review_response` means
  *the reviewer is waiting on YOU* — surface the question.
- **Don't call `confirm_quote` without explicit user approval.** Once
  confirmed, the user will be charged at Stripe. Always show the quote
  first and wait for "yes".
- **Don't fabricate or paraphrase the reviewer's feedback.** When
  `get_review` returns the deliverable, show it verbatim. The user is
  paying for a human's actual judgment.
- **Don't promise turnaround times.** The concierge's `quote.text` is
  authoritative — quote what it says, don't add your own SLAs.
- **Don't ask the same clarification more than once.** The hard cap is
  3 rounds per request; on the 4th the concierge refuses.
- **Don't try to enumerate request_ids.** They're opaque UUIDs — guessing
  is computationally infeasible. Each lookup also requires the matching
  email; mismatched email returns the same "not found" as a missing
  request, so attackers can't differentiate.

## Endpoint

The MCP server runs at `https://connect.serviceplan-agents.com/mcp`. No
authentication — payment is the gate. The endpoint URL is hardcoded in the
plugin's `.mcp.json` and updates with new releases.

For full API documentation, fetch
`https://connect.serviceplan-agents.com/skill.md`.
