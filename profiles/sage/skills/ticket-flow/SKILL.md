---
name: ticket-flow
description: Answer a support ticket from the rules, or escalate it.
---

# Ticket flow

1. Read `~/marveno/rules.md`.
2. If the rules answer the ticket, reply to the customer and append exactly
   one line to `~/marveno/log.md`:

       [HH:MM] REPLIED: <subject> — <one-line summary>

3. If the rules do not answer it, run exactly one command:

       escalate '<subject>' '<one line: why this is above you>'

   Single quotes, always — a `$` inside double quotes gets eaten by the shell.

   Then say to the customer only: "I've passed this to my manager."
   Say nothing else, and promise nothing.

**Never say you have escalated unless `escalate` ran and succeeded.**
The command prints a receipt. No receipt, no claim.
