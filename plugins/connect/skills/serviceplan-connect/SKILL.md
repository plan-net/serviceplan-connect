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

| Tier | Typical use | Time |
|---|---|---|
| **Quick** | Sanity check, fact-correction, gut-check | ~15 min reviewer time |
| **Standard** | Full review with structured pushback memo | ~45 min reviewer time |
| **Deep** | Strategic deep-dive, multiple angles, recommendations | ~2h reviewer time |

The concierge picks the tier automatically based on the brief. Pricing is
shown before any payment is taken.

## Tools (auto-wired via MCP)

You have 6 MCP tools in the `connect` namespace:

1. **`request_review(email, work_summary, ...)`** — start a new request.
   Returns `request_id`. ALWAYS ask the user for their email first if you
   don't have it; it's required for the receipt and follow-up.

2. **`get_request_status(request_id)`** — poll status. Returns one of:
   - `received` (queued — poll again in 30s)
   - `analysing` + `next_question` (concierge needs clarification)
   - `quoted` + `quote` (price + scope description, await user's go-ahead)
   - `awaiting_payment` + `payment_url` (Stripe link, send to user)
   - `paid` (reviewer has been engaged)
   - `delegated` / `reviewing` (work in progress — Phase 3)
   - `completed` + `review_text` (deliverable ready)

3. **`respond_to_clarification(request_id, response)`** — relay the user's
   answer when the concierge asks a clarifying question.

4. **`confirm_quote(request_id)`** — user accepted the quote. Returns a
   Stripe payment URL the user opens in their browser.

5. **`get_review(request_id)`** — fetch the final reviewer feedback once
   status is `completed`. Show it to the user verbatim, then offer to act
   on the suggestions.

6. **`cancel_request(request_id, reason?)`** — bail out at any non-terminal
   state. No charge if status was `quoted` or earlier; refund if paid.

## Recommended interaction flow

```
1. User: "Can someone review this brief?"
2. You: "I can submit it to a senior strategist via Serviceplan Connect.
        I'll need your email for the receipt and the brief itself.
        It costs €50–€1500 depending on depth — you'll see a quote before paying."
3. User shares email + work
4. You call request_review(email, work_summary, attachments=[...])
5. You poll get_request_status every 30s for ~2 min (concierge takes 30-90s)
6. If next_question lands: ask user, call respond_to_clarification, poll again
7. When status='quoted': show the quote to the user verbatim, ask if they want to proceed
8. If user accepts: confirm_quote → show payment URL → tell user to open it
9. After payment, status flips to 'paid' (Phase 2/3) — tell user the reviewer
   has been engaged and you'll let them know when feedback lands
10. (Phase 3) When status='completed', call get_review → show the reviewer's
    feedback, offer to incorporate suggestions
```

## Persisting the request_id (multi-session continuity)

Reviews can take hours or days. **Save the `request_id` somewhere durable**
(notes file, project memory, conversation pin) so the user can come back
later — possibly in a new session, possibly with a different agent — and
fetch the result with just that one number.

The Stripe receipt the user receives also includes the `request_id`
prominently, so they always have a fallback handle.

## Email is the user's primary identifier

Connect does not issue API keys or accounts. The `email` passed to
`request_review` is THE customer identifier — it's used for the receipt,
follow-up, and (Phase 3) the link the user clicks to read the final review.

If the user already used Connect before with the same email, the new
request lands in the same customer record — no duplicate accounts.

## Things to NOT do

- **Don't poll `get_request_status` faster than every 30s.** The IP rate
  limit is 10 requests/hour/IP; chatty polling will lock you out.
- **Don't call `confirm_quote` without explicit user approval.** Once
  confirmed, the user will be charged at Stripe (Phase 2 — currently a
  placeholder). Always show the quote first and wait for "yes".
- **Don't fabricate or paraphrase the reviewer's feedback.** When
  `get_review` returns the deliverable, show it verbatim. The user is
  paying for a human's actual judgment.
- **Don't promise turnaround times.** The concierge gives an estimate in
  the quote text — quote that estimate, but don't add your own.

## Endpoint

The MCP server runs at `https://connect.sumike.ai/mcp` (currently staging
during early access). No authentication — payment is the gate. The endpoint
URL is hardcoded in the plugin's `.mcp.json` and updates with new releases.

For full API documentation including raw JSON-RPC schemas, fetch
`https://connect.sumike.ai/skill.md`.
