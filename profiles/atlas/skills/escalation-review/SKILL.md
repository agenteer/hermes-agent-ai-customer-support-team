---
name: escalation-review
description: Decide one escalation, or ask the operator to decide.
---

# Escalation review

1. Read `~/marveno/rules.md`.
2. If the rules let you decide, decide and run:

       resolve '<subject>' '<your decision, one line>'

   Single quotes, always — a `$` inside double quotes gets eaten by the shell.

3. If the rules say nobody decides alone, run:

       ask-operator '<subject>' '<the situation, the rule it hits, and your recommendation with options>'

   Then stop. Do not decide, and do not run `resolve`. The operator's answer
   will arrive later as a new message.

4. A message from the operator is their decision on the newest
   `PENDING OPERATOR` line in `~/marveno/log.md` — run `resolve` with it.

You decide (or relay the operator's decision). You do not write to the
customer; `resolve` hands that to Sage.
