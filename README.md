# Hermes Agent AI customer support team tutorial — companion files

The companion files for **Build an AI Customer Support Team That Escalates to You, Using Hermes Agent**: three Hermes Agent profiles on one server — the default one Hermes installs, plus two you create — run customer support for a fictional company, Marveno — a front desk that routes what comes in, a support specialist (Sage) that answers from a rules file, and a manager (Atlas) that reviews escalations and brings the decisions that are yours to you in Telegram. Customers reach the team by email; replies land in their own thread.

- Article: https://agenteer.com/learn/tutorials/hermes-agent-ai-customer-support-team/
- Video: https://youtu.be/VgmovF822JQ
- A free server for it (Oracle always-free tier), click by click: https://agenteer.com/learn/tutorials/hermes-agent-oracle-cloud/

Built on Hermes Agent v0.20.4 (2026.8.18), himalaya v2.1.0, Ubuntu 24.04. Newer versions will differ in small ways; the article says which checks to re-run.

## What is here

| Path | What it is | Where it goes on your server |
| --- | --- | --- |
| `profiles/sage/SOUL.md` · `profiles/atlas/SOUL.md` | Each agent's identity file | `~/.hermes/profiles/<name>/SOUL.md` |
| `profiles/sage/config.yaml` · `profiles/atlas/config.yaml` | Each agent's model, two lines | `~/.hermes/profiles/<name>/config.yaml` |
| `gateway-block.yaml` | The front desk's routing block as of step 5: one listener, three routes, the Telegram home address (the catch-all route and the email address are added in steps 5 and 8 — see `COMMANDS.md`) | appended to `~/.hermes/config.yaml` |
| `env.example` | The keys the gateway reads, with placeholders | `~/.hermes/.env` |
| `rules.md` | The company on one page: what we can say, who decides what | `~/marveno/rules.md` |
| `profiles/sage/skills/ticket-flow/SKILL.md` | Sage's procedure | `~/.hermes/profiles/sage/skills/ticket-flow/SKILL.md` |
| `profiles/atlas/skills/escalation-review/SKILL.md` | Atlas's procedure | `~/.hermes/profiles/atlas/skills/escalation-review/SKILL.md` |
| `commands/` | Five small commands that move work and print receipts, in six files (`resolve` has a Telegram-only version and a finished one) | `~/.local/bin/` |
| `himalaya/config.example.toml` | Fallback mail-client config (the wizard normally writes this) | `~/.config/himalaya/config.toml` |
| `COMMANDS.md` | The commands of the build, in order, copy-pasteable | — |

The token, the App Password, the mail addresses, and the group and room numbers are placeholders. Step 5 in `COMMANDS.md` fills in the Telegram numbers; the email values go in at step 8.

## How to use it

On the server (the one with Hermes installed):

```bash
git clone https://github.com/agenteer/hermes-agent-ai-customer-support-team.git ~/hermes-agent-ai-customer-support-team
cd ~/hermes-agent-ai-customer-support-team
cat COMMANDS.md
```

`COMMANDS.md` is the whole build in order; the article explains each step. The steps and the files they use:

| Step | What you do | Files from this repo |
| --- | --- | --- |
| 1 | A server that stays on, Hermes installed | — |
| 2 | Sign in once; meet the default agent; create Sage and Atlas | `profiles/*/SOUL.md`, `profiles/*/config.yaml` |
| 3 | How the team talks (no commands — the map) | — |
| 4 | Telegram: one bot, one group, three rooms; read the numbers | — |
| 5 | Wire the front desk: keys, routes, start the gateway | `env.example` (Telegram lines), `gateway-block.yaml` |
| 6 | The rules file | `rules.md` |
| 7 | Skills and the four commands; the first internal loop | `profiles/*/skills/`, `commands/escalate`, `commands/ask-operator`, `commands/resolve.telegram-only`, `commands/tell-customer` |
| 8 | Email both directions; the fifth command; teach `resolve` the second door | `env.example` (EMAIL lines), `commands/tell-customer-email`, `commands/resolve`, `himalaya/` |
| 9 | Run the team: three tickets, one per tier; the routine | — |

Two things about the order — Sage's `SOUL.md` names a skill that arrives in step 7, and `resolve` comes in two versions — are explained where they apply in `COMMANDS.md` (steps 2, 7, and 8).

## License

MIT — see `LICENSE`.
