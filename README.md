# Serviceplan Connect

> Request human-expert review of AI agent work, mid-task.

**Serviceplan Connect** lets AI agents — your Cursor, Claude Code, Claude.ai, or
any custom agent — pause and ask a senior human at [Serviceplan](https://serviceplan.com)
to review the work before you act on it. Submit the brief, get a priced quote
in 30-90 seconds, pay, and a real strategist or creative director takes over
from there. Their feedback flows back to your agent.

It's for the moment when your AI has done good work but you want a human's
judgment before the meeting, the pitch, the press release, the C-suite ask.

---

## How it works

1. **Your agent submits the brief** — what was done, what to review, your email
2. **The concierge analyses + quotes** — Quick / Standard / Deep tier with a price
3. **You approve, pay** — via Stripe (EUR, with VAT for EU customers)
4. **A Serviceplan expert reviews** — 15 min to 2 hours of focused human time
5. **Feedback flows back to your agent** — your agent can show you the review
   and help you act on it

No accounts. No API keys. Email is your identifier; the request ID is your
durable handle. Anyone whose AI agent supports MCP can use this.

---

## Install

### For Claude Code (recommended)

Install as a plugin in two commands:

```bash
/plugin marketplace add plan-net/serviceplan-connect
/plugin install connect@serviceplan-connect
```

That's it. Claude Code wires up the MCP server, loads the skill, and your
next session knows when and how to use Connect. Try it: *"can someone
review this?"*

### For other AI agents (Cursor, Claude.ai, custom)

Point your agent at the hosted skill description — any agent that can fetch
a URL and follow markdown instructions will figure out the rest:

> "Read https://connect.sumike.ai/skill.md and help me get a review of [your work]."

The skill explains the 6 MCP tools, the polling model, and the recommended
interaction flow. Behind the scenes the agent calls
`https://connect.sumike.ai/mcp` (open MCP, no auth — payment via Stripe is
the gate).

---

## What you'll be charged

| Tier | What you get | Typical price |
|---|---|---|
| **Quick** | Sanity check, fact correction, tight gut-check | €50 |
| **Standard** | Full structured review with prioritized critiques | €500 |
| **Deep** | Strategic deep-dive with multiple angles + recommendations | €1500 |

The concierge picks the tier automatically based on the brief. You see the
quote (with text describing exactly what the reviewer will deliver) before
any payment is taken. You can always cancel before paying.

VAT is added at checkout based on your billing address. EU B2B customers
can enter their VAT ID for reverse charge.

---

## Status

**Phase 1 — early access (current)** — concierge intake, quoting, polling all
work end-to-end on staging. Stripe payment is wired but in test mode (no
real money moves). Use the staging endpoint `connect.sumike.ai/mcp`.

**Phase 2 — Stripe live (in progress)** — real payment, EUR, VAT, Stripe Tax.
Production endpoint `connect.serviceplan-agents.com/mcp` will go live.

**Phase 3 — reviewer delegation (planned)** — once paid, the concierge selects
a reviewer (Sebastian for v1, plus more), packs a briefing, sends a
delegation email, relays the response back. The full loop closes.

**Phase 4 — Agent Messenger channel (planned)** — same backend, accessible
via DM to a Masumi agent slug instead of MCP.

Track progress at [the source repo](https://github.com/plan-net/agentic-coworkers).

---

## What's in this repo

```
.claude-plugin/marketplace.json    # Claude Code marketplace listing
plugins/connect/
  ├── .claude-plugin/plugin.json   # Plugin manifest
  ├── .mcp.json                    # MCP server config (HTTP, no auth)
  └── skills/
      └── serviceplan-connect/
          └── SKILL.md             # Skill teaching Claude when + how to use Connect
```

The MCP server itself is hosted by Plan.Net Studios — not in this repo. This
repo is purely the distribution package: the marketplace manifest + plugin
manifest + skill definition + this README.

---

## License

MIT — see [LICENSE](./LICENSE). Use it however you want.

The Serviceplan Connect service itself (the hosted MCP backend, the
concierge agent, the reviewer marketplace) is operated by [Plan.Net
Studios](https://plan.net) under [Serviceplan Group](https://serviceplan.com).

---

## Built by

[Plan.Net Studios](https://plan.net) — agentic systems for the
Serviceplan Group.
