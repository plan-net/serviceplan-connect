---
name: serviceplan-connect
version: 0.2.0
description: "Request human-expert review of AI agent work — Serviceplan Connect concierge handles intake, quoting, payment, and delegation. Polling-based MCP, no auth required."
homepage: https://connect.serviceplan-agents.com
metadata:
  category: human-in-the-loop
  api_base: https://connect.serviceplan-agents.com/mcp
  payment_required: true
  channels: [mcp]
  interaction_model: polling
---

# Serviceplan Connect

You're helping a user get **human-expert review** of work you (or another agent)
has done. Serviceplan Connect is a paid service: a backend concierge agent
analyses the request, returns a priced quote, takes payment, and delegates
to a Serviceplan expert. The expert's feedback flows back to you.

This skill tells you how to use it.

---

## 🧩 If you are Claude Code: install the official plugin

There's a first-class Claude Code plugin at
[**`plan-net/serviceplan-connect`**](https://github.com/plan-net/serviceplan-connect).
Tell the user to run two commands and they get the MCP server + a richer
skill auto-installed:

```
/plugin marketplace add plan-net/serviceplan-connect
/plugin install connect@serviceplan-connect
```

After install, the `connect` MCP tools become available in every session
without any further setup, and a tighter SKILL.md activates automatically
when the user asks for review. **Recommend this path to Claude Code users
before the manual MCP route below** — it's two commands, lasts across
sessions, and updates with new releases.

For any other agent (Cursor, Claude.ai, custom): keep reading. The MCP
endpoint is open and works the same.

---

## ⏱ Interaction model: polling (read this first)

Connect runs the concierge in a stateful chat session on the backend. **Your
MCP tool calls don't block while the concierge thinks.** They return
immediately with a `request_id`, and you poll `get_request_status` to see
what the concierge has produced.

```
Turn 1:  request_review(...)              → returns {request_id, status:"received", poll_after_seconds:30}
Wait ~30s
Turn 2:  get_request_status(request_id)   → returns either:
                                              {status:"quoted",   quote:{...}, action_required:"confirm_quote"}
                                              {status:"analysing", next_question:"...", action_required:"answer_clarification"}
                                              {status:"received",  message:"still queued, poll again"}
Repeat as needed.
```

**Why polling:** the concierge takes 30-90 seconds to think (Opus model
reasoning over the work_summary). Some flows have a clarification round-trip,
which can add minutes-to-hours of customer-side wait. A few have human-expert
delivery a day or two later. None of that fits a synchronous tool call. The
polling pattern lets you keep the user's experience smooth — show "I've
submitted your request, checking back in 30 seconds…" and update when status
changes.

### 🔑 Persist the `request_id`

The `request_id` is the **only handle** to a customer's review. Save it:

- **Within the conversation:** keep `request_id` in your context so follow-up
  user prompts ("did the review come back?") can call `get_review` directly
  without asking the user to repeat themselves.
- **Across sessions / agent restarts:** if you have any persistent memory
  (Cursor's memory, a per-user knowledge file, etc.), record `request_id`
  alongside the user's email so a future session can pick up where they
  left off.
- **Tell the user the `request_id`** explicitly when you submit a request.
  We also send a notification email to the user's address when the human
  reviewer's response is ready (Phase 3) — that email includes the
  `request_id`. The user can paste it back into a fresh agent conversation
  to retrieve their review days later.

---

## Quick start (copy-paste-ready loop)

```
1. Ask the user for their email (REQUIRED).
2. Optional but useful: ask for their name + organization.
3. Call connect.request_review(email, work_summary, attachments?, name?, org?, urgency?)
   → returns {request_id, status:"received", poll_after_seconds:30, message}
   → TELL THE USER THE request_id ("Submitted as Connect request #42; I'll check back in 30s.")
4. Wait ~30 seconds. Then call connect.get_request_status(request_id).
5. Branch on the response:
   - status="received" or "analysing" without next_question → wait + poll again
   - status="analysing" with next_question                  → ask the user, call respond_to_clarification
   - status="quoted"                                        → show the quote, ask for confirmation
6. If the user accepts, call connect.confirm_quote(request_id) → returns payment URL (or
   Phase-1 placeholder).
7. Once Phase 3 ships: poll get_request_status occasionally (or wait for the user to bring
   it up). When status="completed", call get_review(request_id) and show the response.
```

---

## Tools

All tools are exposed at `https://connect.serviceplan-agents.com/mcp`. The endpoint is
**open — no authorization required**. Payment via Stripe (Phase 2) is the
gate, not pre-issued credentials.

### `connect.request_review`

Submit a new request.

| Field | Required | Notes |
|---|---|---|
| `email` | yes | User's email. Used to track the request and follow up. |
| `work_summary` | yes | What the user wants reviewed. Be specific: include what was done, what question they want answered, and why a human review matters. |
| `attachments` | no | List of `{name, mime, url}` (preferred — URL points to user's storage) or `{name, mime, inline_b64}` (≤256KB). Use URLs from the user's storage when possible. |
| `name` | no | User display name. |
| `org` | no | User's organization. |
| `urgency` | no | `'low' \| 'normal' \| 'high'` |

Returns: `{request_id, status, poll_after_seconds, message}`.

**Wait at least 30 seconds before polling status** — concierge reasoning
takes 30-90s on average.

### `connect.get_request_status`

Read-only status. The shape varies by current `status`:

| status | What's in the response | What you should do |
|---|---|---|
| `received` | `message`, `poll_after_seconds` | Wait, poll again |
| `analysing` | `message`, `poll_after_seconds`. May include `next_question` and `action_required="answer_clarification"`. | If `next_question` present → ask user, call `respond_to_clarification`. Else wait + poll. |
| `quoted` | `quote: {tier, amount_usd, text, valid_until}`, `action_required="confirm_quote"` | Show quote (use `quote.text` verbatim), ask user to confirm |
| `awaiting_payment` | `message` (Phase 1: out-of-band follow-up; Phase 2: includes `payment_url`) | Show payment URL when present |
| `paid` / `delegated` / `reviewing` | `message` ("human reviewer is handling this") | Tell user it's with the reviewer; they'll get an email when ready |
| `completed` | `action_required="fetch_review"`, `message` | Call `get_review` and show the response |
| `cancelled` / `expired` / `payment_failed` / `refunded` | `message` | Terminal — no further calls valid |

### `connect.respond_to_clarification`

Pass the user's answer to the concierge's `next_question`.

| Field | Required |
|---|---|
| `request_id` | yes |
| `response` | yes |

After answering, poll `get_request_status` again (typical wait: 20-60 seconds
for the concierge to take the follow-up turn).

### `connect.confirm_quote`

User accepts the quote — moves status to `awaiting_payment`. **Phase 1 returns
`payment_url=null`**; the Serviceplan team follows up out-of-band. Phase 2
will return a real Stripe Checkout URL — show it to the user, they pay
out-of-band, the request automatically advances when the webhook fires.

### `connect.get_review`

Fetch the expert's response once `status="completed"`. Idempotent — safe to
call from a future session days later, as long as you have the `request_id`.

### `connect.cancel_request`

Cancel a request from any non-terminal state.

| Field | Required |
|---|---|
| `request_id` | yes |
| `reason` | no |

---

## Pricing tiers (current)

The concierge picks the tier based on the work's complexity. Show the user
the `quote.text` the concierge generated — don't reinterpret pricing.

| Tier | Roughly | Default price (USD) |
|---|---|---|
| Quick | ~15 min sanity check | $50 |
| Standard | ~45 min full review | $150 |
| Deep | ~2h strategic deep-dive | $400 |

---

## Etiquette

- **Always collect the user's email first.** It's the primary identifier
  and the address we use for cross-session notifications.
- **Tell the user their `request_id`.** It's their handle to come back to
  this review later from a different agent session.
- **Show the `quote.text` verbatim** to the user. Don't paraphrase pricing.
- If the concierge asks a clarifying question, **ask the user** — don't make
  up an answer.
- Once the user accepts and you've called `confirm_quote`, give the user
  the `request_id` again so they can check back later via `get_review`.
- **Don't poll faster than every 30 seconds.** The concierge needs time
  to think. Polling faster wastes our budget and yours.
- **Rate limit:** 10 requests/hour per IP on the open endpoint. If you get
  HTTP 429, back off and try again in an hour.

---

## What a great `work_summary` looks like

Bad: "Review my analysis"

Good:
> "I just produced a competitive analysis of the German EV charging market for
> a Plan.Net Studios client. I focused on Tesla, Ionity, and EnBW. The client
> wants me to confirm whether my read on Tesla's pricing strategy in Germany
> is defensible — I argued they're testing premium positioning but the data is
> thin. Could a Serviceplan partner sanity-check this section before I send it
> tomorrow morning?"

The concierge needs enough context to pick a tier and write a useful brief
for the human reviewer. Vague summaries get clarification questions, which
slow things down.

---

## Lifecycle scenarios you should know about

**Scenario A — Quote → confirm → review (single session):**
The whole loop happens within minutes. User accepts the quote, pays
(Phase 2), reviewer responds (Phase 3) — you see status flip to `completed`
on a periodic poll, then call `get_review`.

**Scenario B — Cross-day clarification:**
Concierge asks a clarification, user says "let me think about it" and goes
home. The next day, in a fresh agent session, the user provides the answer.
You'll need the `request_id` from somewhere — your saved memory, the
notification email, or the user remembers it. Call
`respond_to_clarification(request_id, answer)` and the concierge picks up
where it left off (cold session resume — adds ~5s to the next poll).

**Scenario C — Days-later review fetch:**
User submits a Deep review, walks away. A day or two later, the human
expert responds. We email the user a notification with the `request_id`
and a link. User can come back via any agent (yours, another, a different
device) and call `get_review(request_id)` to see the response.
