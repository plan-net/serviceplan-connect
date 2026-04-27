# Serviceplan Connect

## The agency your agents would hire.

The next decade of work is **AI-first**. Agents — yours, ours, anyone's — do more of the doing every quarter. But the best work has always come from a sparring match between sharp people and sharp tools. Models keep getting better. Senior humans keep getting rarer. Where the two meet on eye level — that's where the magic still happens.

**Serviceplan Connect** is the on-demand spar partner for your agents. Whenever an agent needs a real strategist, creative director, or account lead to push back, sharpen, or sign off on its work — Connect makes that human one tool call away. Backed by [Serviceplan](https://serviceplan.com), one of Europe's largest independent agencies.

> **Private beta.** No published SLAs yet — we'll publish them as the network scales. Pricing, scope, and turnaround are agreed per request before anything is charged.

---

## Add Connect to your agent in one sentence

Paste this into [Claude Code](https://claude.com/claude-code), [Cursor](https://cursor.sh), [Claude.ai](https://claude.ai), or any agent that speaks MCP:

> *"Read https://connect.serviceplan-agents.com/skill.md and set yourself up to work with Serviceplan Connect."*

Your agent fetches the skill, wires up the MCP connection, and is ready to commission human reviews from then on. No installs, no accounts, no API keys — email is the user's identity.

For Claude Code specifically there's a [first-class plugin install](#install-as-a-claude-code-plugin) below — two commands.

---

## Why we built this

Models are extraordinary at first drafts. They're getting better at second drafts. What they don't have, and won't have any time soon, is **twenty years of having watched campaigns ship, fail, and ship again** — the lived pattern recognition that lets a senior catch the angle the brief doesn't ask for.

Most marketing teams now have an AI in the loop ([Claude Cowork](https://serviceplan-agents.com), Cursor, ChatGPT, in-house agents). The work that comes out is real. But before it goes to the C-suite, the client meeting, the keynote — there's nobody around to stress-test it. No senior who's seen this play out fifty times. Stuff ships at "good enough." Sometimes that's fine. Sometimes it's the difference between a campaign and a missed quarter.

Connect closes that gap. The agent does the heavy lift. A senior human at Serviceplan brings the pattern recognition. **Best models, best humans, on eye level.** That's the whole pitch.

The reviewers themselves are AI-augmented — they use [our own coworkers](https://serviceplan-agents.com) to gather context, structure responses, and ship faster. But the **judgment, the pushback, and the stamp of approval are unmistakably human**. Serviceplan stands behind what the reviewer says.

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
5. ─────────────────────────────────────►        a senior reviewer is engaged,
                                                  AI-supported but human-led
                                                       │
                                                       │ (15 min – half day
                                                       │  of focused
                                                       │  human attention)
                                                       │
6. polls for review                    ◄─────    structured feedback,
                                                  signed off by a real human
7. you act on it
```

### Tiers

| Tier | What you get | Reviewer time | Indicative price |
|---|---|---|---|
| **Quick** | Tight sanity check on a single artifact. One specific question, one specific answer. | ~15 min | €50 |
| **Standard** | Full structured review of a non-trivial deliverable. Multi-section pushback. | ~45 min | €150 |
| **Pro** | Strategic review with substantial context. Multiple options weighed, structured response. | ~2 h | €400 |
| **Deep** | Senior partner engagement. Multi-angle deep-dive, written recommendations, fit for executive consumption. | ~½ day | €1500 |

Indicative — the concierge picks the tier and gives you the exact price + a description of what the reviewer will deliver before any payment is taken. You can cancel anytime before paying.

---

## Integration channels

| Channel | Status | For whom |
|---|---|---|
| **MCP** (Model Context Protocol) | ✅ Live | Claude Code, Cursor, Claude.ai, any MCP-capable agent |
| **Agent Messenger** (A2A direct messaging) | 🛠 Planned | Autonomous agents that talk to other agents — DM the `connect-concierge` slug |

The protocol is open and intentionally minimal. We don't care what your agent's stack is.

---

## Payment

| Method | Status | When to use it |
|---|---|---|
| **Invoice** | ✅ Existing Serviceplan customers | Roll Connect into your existing account, pay on net terms |
| **Stripe** | 🛠 Coming | Personal credit card or company card, one-off |
| **x402** | 🛠 Planned | Direct agent-to-agent payment for fully-autonomous workflows |

EUR primary. VAT handled per country. EU B2B reverse charge supported via VAT ID.

---

## Install as a Claude Code plugin

Two commands:

```
/plugin marketplace add plan-net/serviceplan-connect
/plugin install connect@serviceplan-connect
```

After install, the `connect` MCP tools are wired into every session and a tighter skill activates automatically when you say *"can someone review this?"* — no need to paste the bootstrap sentence above.

To update later: `/plugin update connect@serviceplan-connect` (auto-update is enabled by default for marketplace plugins).

---

## Status

**Private beta.** What that means in practice:

- We're actively iterating on the concierge prompts, pricing, and reviewer-matching logic
- No published response-time SLAs yet — the concierge gives an estimate per quote
- Reviewer network is small and growing
- Feedback drives the roadmap. If you try it, [open an issue](https://github.com/plan-net/serviceplan-connect/issues) — we read every one

---

## What's in this repo

This repository is the public hub for everything an agent (or a developer building agentic tooling) needs to integrate with Serviceplan Connect. The Claude Code plugin is the first artifact; more interfaces, integrations, and reference clients land here over time.

```
.claude-plugin/marketplace.json    # Claude Code marketplace listing
plugins/connect/
  ├── .claude-plugin/plugin.json   # Plugin manifest
  ├── .mcp.json                    # MCP server config
  └── skills/serviceplan-connect/
      └── SKILL.md                 # Skill teaching Claude when + how to use Connect
```

The MCP server itself is hosted by Plan.Net Studios — not in this repo. This repo is purely the public distribution package.

---

## License

MIT — see [LICENSE](./LICENSE). Use it however you want.

The Serviceplan Connect service itself (the hosted backend, the concierge, the reviewer network) is operated by [Plan.Net Studios](https://plan.net) under [Serviceplan Group](https://serviceplan.com).

---

Built by [Plan.Net Studios](https://plan.net) — agentic systems for the Serviceplan Group.
