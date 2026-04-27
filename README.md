# Serviceplan Connect

## The agency your agents would hire.

We believe the future of work is **AI-first**. Your agents do the doing — research, drafts, decks, briefs, plans. But every now and then a piece of work needs to be more than *good enough*. It needs a senior human to look at it, push back, and put their name on it.

**Serviceplan Connect** lets your AI agent commission that human review on demand — from a real network of marketing strategists, creative directors, and account leads at [Serviceplan](https://serviceplan.com), one of Europe's largest independent agencies.

> **Private beta.** No SLAs yet — we'll publish them as the network scales. Pricing, scope, and turnaround are agreed per request before anything is charged.

---

## Get started in one sentence

Paste this into [Claude Code](https://claude.com/claude-code), [Cursor](https://cursor.sh), [Claude.ai](https://claude.ai), or any agent that speaks MCP:

> *"Read https://connect.sumike.ai/skill.md and help me get a senior reviewer to look at [your work]."*

Your agent fetches the skill, wires up the MCP connection, walks you through brief → quote → review. No installs, no accounts, no API keys. Email is your identity.

For Claude Code specifically, there's a [first-class plugin install](#install-as-a-claude-code-plugin) below — two commands.

---

## Why we built this

The pattern keeps repeating: a brand strategist or marketing lead spends an evening with [Claude Cowork](https://serviceplan-agents.com), produces a tight positioning brief, a competitive read, a campaign concept. The AI work is genuinely good. But before it goes to the C-suite, the client meeting, the press release — there's nobody to push back. No senior who's seen this play out fifty times before. No reviewer who'll catch the angle the AI missed because no AI has lived through it.

That's the gap. Connect closes it.

**The flip:** today, humans use AI as a tool. Tomorrow, agents are the primary workers — and agents need access to *agencies of human expertise* the same way they need access to APIs. Connect is that access layer for Serviceplan.

---

## How it works

```
Your agent                                       Serviceplan Connect
─────────                                        ───────────────────
1. submits brief + your email          ─────►    concierge analyses (Opus)
                                                       │
2. polls for status                    ◄─────    quote: tier + price + scope
                                                       │
3. shows you the quote                                 │
4. you accept                          ─────►    payment (invoice / card / agent)
                                                       │
5. ─────────────────────────────────────►        a senior reviewer is engaged
                                                       │
                                                       │ (15 min – 2 h
                                                       │  of human time,
                                                       │  often supported
                                                       │  by Serviceplan's
                                                       │  own AI coworkers)
                                                       │
6. polls for review                    ◄─────    structured feedback,
                                                  stamped by a real human
7. you act on it
```

The reviewer is supported by AI — our own coworkers at [serviceplan-agents.com](https://serviceplan-agents.com) help them gather context, structure feedback, and ship faster. But the **judgment, the pushback, and the stamp of approval are human**. You can hold Serviceplan accountable for what the reviewer says.

### Tiers

| Tier | What you get | Typical price |
|---|---|---|
| **Quick** | Sanity check, fact correction, gut check | €50 |
| **Standard** | Full structured review, prioritized critiques | €500 |
| **Deep** | Strategic deep-dive, multi-angle, recommendations | €1500 |

The concierge picks the tier. You see the quote — text describing exactly what the reviewer will deliver — before any payment is taken. You can cancel anytime before paying.

---

## Integration channels

| Channel | Status | For whom |
|---|---|---|
| **MCP** (Model Context Protocol) | ✅ Live | Claude Code, Cursor, Claude.ai, any MCP-capable agent |
| **Masumi Agent Messenger** | 🛠 Phase 4 | Agents on the [Masumi](https://masumi.network) network — DM the `connect-concierge` slug |

The protocol is open and intentionally minimal. We don't care what your agent's stack is.

---

## Payment

| Method | Status | When to use it |
|---|---|---|
| **Invoice** | ✅ Existing Serviceplan customers | Roll Connect into your existing account |
| **Stripe** | 🛠 Phase 2 (currently test mode on staging) | Personal credit card, one-off |
| **x402** | 🛠 Planned | Direct agent-to-agent payment for fully-autonomous workflows |

EUR primary. VAT handled per country (Stripe Tax). EU B2B reverse charge supported via VAT ID.

---

## Install as a Claude Code plugin

Two commands:

```
/plugin marketplace add plan-net/serviceplan-connect
/plugin install connect@serviceplan-connect
```

After install, the `connect` MCP tools are wired into every session, and a tighter skill activates automatically when you say *"can someone review this?"* — no need to paste the one-sentence bootstrap above.

To update later: `/plugin update connect@serviceplan-connect` or just restart your session (auto-update is enabled by default for marketplace plugins).

---

## Status

**Private beta.** What that means in practice:

- We're actively iterating on the concierge prompts, pricing rubric, and reviewer-matching logic
- No published response-time SLAs yet — the concierge gives an estimate per quote
- Production endpoint (`connect.serviceplan-agents.com`) goes live with Phase 2 (real Stripe). Until then, early access points at the staging endpoint (`connect.sumike.ai`)
- We're collecting feedback on the setup experience, the agent-side UX, and the reviewer-side workflow. If you try it, [open an issue](https://github.com/plan-net/serviceplan-connect/issues) — we read every one

The full roadmap and source code live in the [agentic-coworkers](https://github.com/plan-net/agentic-coworkers) repo (private).

---

## What's in this repo

```
.claude-plugin/marketplace.json    # Claude Code marketplace listing
plugins/connect/
  ├── .claude-plugin/plugin.json   # Plugin manifest
  ├── .mcp.json                    # MCP server config (HTTP, no auth)
  └── skills/serviceplan-connect/
      └── SKILL.md                 # Skill teaching Claude when + how to use Connect
```

The MCP server itself is hosted by Plan.Net Studios — not in this repo. This repo is purely the distribution package.

---

## License

MIT — see [LICENSE](./LICENSE). Use it however you want.

The Serviceplan Connect service itself (the hosted MCP backend, the concierge, the reviewer network) is operated by [Plan.Net Studios](https://plan.net) under [Serviceplan Group](https://serviceplan.com).

---

Built by [Plan.Net Studios](https://plan.net) — agentic systems for the Serviceplan Group.
