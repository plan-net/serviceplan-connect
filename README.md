# Serviceplan Connect

## The agency your agents would hire.

When your AI agent needs a real human at [Serviceplan](https://serviceplan.com) to review, push back on, or sign off on its work — Connect makes that human one tool call away.

> **Private beta.** Pricing and scope are confirmed per request before anything is charged. Response-time SLAs per tier are coming as the network scales.

---

## Add Connect to your agent in one sentence

Paste this into [Claude Code](https://claude.com/claude-code), [Cursor](https://cursor.sh), [Claude.ai](https://claude.ai), [Microsoft Copilot Studio](https://copilotstudio.microsoft.com), or any agent that speaks MCP:

> *"Read https://connect.serviceplan-agents.com/skill.md and set yourself up to work with Serviceplan Connect."*

Your agent fetches the skill, wires up the MCP connection, and is ready to commission human reviews from then on. No installs, no accounts, no API keys — email is the user's identity.

Prefer to do it by hand? Skip to [Setup](#setup) below.

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
                                                       │ (review work happens —
                                                       │  scope and turnaround
                                                       │  agreed in the quote)
                                                       │
6. polls for review                    ◄─────    structured feedback,
                                                  signed off by a real human
7. you act on it
```

### Tiers

Pricing is **value-based** — what is it worth to have a senior human look at this? — not time-based. The concierge picks the tier and gives you the exact price + a description of what the reviewer will deliver before anything is charged. You can cancel anytime before paying.

| Tier | What you get | Indicative price |
|---|---|---|
| **Quick** | A senior eye on a single, well-scoped artifact. One question in, one tight answer out. | €50 |
| **Standard** | A structured pushback memo on a non-trivial deliverable. Multi-section critique with prioritized fixes. | €150 |
| **Pro** | A strategic review with substantial context. Multiple options weighed, recommendations sequenced. | €400 |
| **Deep** | A senior partner engagement. Multi-angle deep-dive with written recommendations, fit for executive consumption. | €1500 |

> Response-time SLAs per tier ship as we scale the reviewer network.

---

## Why we built this

Models are extraordinary at first drafts. They're getting better at second drafts. What they don't have, and won't have any time soon, is **twenty years of having watched campaigns ship, fail, and ship again** — the lived pattern recognition that lets a senior catch the angle the brief doesn't ask for.

Most marketing teams now have an AI in the loop ([Claude Cowork](https://serviceplan-agents.com), Cursor, ChatGPT, in-house agents). The work that comes out is real. But before it goes to the C-suite, the client meeting, the keynote — there's nobody around to stress-test it. No senior who's seen this play out fifty times. Stuff ships at "good enough." Sometimes that's fine. Sometimes it's the difference between a campaign and a missed quarter.

Connect closes that gap. The agent does the heavy lift. A senior human at Serviceplan brings the pattern recognition. **Best models, best humans, on eye level.** That's the whole pitch.

The reviewers themselves are AI-augmented — they use [our own coworkers](https://serviceplan-agents.com) to gather context, structure responses, and ship faster. But the **judgment, the pushback, and the stamp of approval are unmistakably human**. Serviceplan stands behind what the reviewer says.

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

## Setup

### The lazy way: ask your agent

For any agent that can fetch URLs and follow markdown instructions (Claude.ai, Cursor, Claude Code, Microsoft Copilot Studio, custom agents), paste the [bootstrap sentence](#add-connect-to-your-agent-in-one-sentence) at the top of this README. The agent fetches `https://connect.serviceplan-agents.com/skill.md` and figures out how to wire itself up.

### Claude Code (one-click plugin)

```
/plugin marketplace add plan-net/serviceplan-connect
/plugin install connect@serviceplan-connect
```

MCP server + skill installed in two commands. Auto-updates on session start. Update manually with `/plugin update connect@serviceplan-connect`.

### Manual setup — pick your agent

Connect's MCP server is **open** (no auth, no API key). All you need to do is point your agent at it.

#### Claude.ai (web app)

1. Go to **Settings → Connectors → Add custom connector**
2. **URL:** `https://connect.serviceplan-agents.com/mcp`
3. Leave OAuth fields empty. Save.
4. In any chat, click the **+** button → **Connectors** → toggle **Connect** on
5. *Optional but recommended:* in your project's **Custom instructions**, paste *"Read https://connect.serviceplan-agents.com/skill.md before responding when the user asks for a human review."*

#### Microsoft Copilot Studio

1. In your agent, go to **Tools → Add a tool → New tool → Model Context Protocol**
2. Fill in:
   - **Server name:** `Serviceplan Connect`
   - **Server description:** `Request human-expert review of AI agent work via Serviceplan Connect`
   - **Server URL:** `https://connect.serviceplan-agents.com/mcp`
3. **Authentication:** None
4. Click **Create → Add to agent**
5. *Optional but recommended:* in your agent's **Instructions** field, paste the contents of `https://connect.serviceplan-agents.com/skill.md` so the agent knows when and how to use the new tools

> Copilot Studio supports the Streamable HTTP MCP transport (the modern standard). Any plan that includes Copilot Studio agent authoring works.

#### Cursor

Add to `~/.cursor/mcp.json` (global) or `.cursor/mcp.json` (project-local):

```json
{
  "mcpServers": {
    "connect": {
      "url": "https://connect.serviceplan-agents.com/mcp"
    }
  }
}
```

*Optional but recommended:* append the contents of `https://connect.serviceplan-agents.com/skill.md` to your project's `.cursorrules` (or `AGENTS.md`).

#### Any other MCP-capable agent

1. Add an HTTP MCP server pointing at `https://connect.serviceplan-agents.com/mcp` (no auth)
2. Make `https://connect.serviceplan-agents.com/skill.md` available to the agent — either as a one-off fetch the agent does at session start, or pasted into the system prompt / instructions / rules file your agent uses

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
