# The commands, in the build's order

Copy-paste list. Placeholders in `<ANGLE_BRACKETS>` are yours to fill; the `sed` line in step 5 fills the Telegram numbers. Built on Hermes Agent v0.20.4 (2026.8.18), himalaya v2.1.0, Ubuntu 24.04. The article explains what each line does and why; this file is the build checklist.

## 1 · Server and install

A computer that stays on, with Ubuntu 24.04. Free option: the Oracle always-free walkthrough — https://agenteer.com/learn/tutorials/hermes-agent-oracle-cloud/ (stop after SSH works; the rest is below).

```bash
sudo apt-get update && sudo apt-get install -y xz-utils libatomic1
curl -fsSL https://hermes-agent.nousresearch.com/install.sh | bash
source ~/.bashrc
hermes --version        # want: a version banner (v0.20.4 here)
```

The installer takes ~10 minutes. Its optional browser download can stall for up to 10 minutes and then warn and continue; this build does not use it. Telegram support is not in the base install; the gateway installs it on first start (step 5).

## 2 · Sign in, meet the default agent, hire the team

```bash
ls ~/.hermes                                   # the default agent's home already exists
hermes auth add openai-codex                   # prints a URL + code; finish the sign-in on your phone or laptop
hermes auth status openai-codex                # want: logged in
hermes setup model                             # the model page of the wizard: OpenAI ▸ ChatGPT/Codex ▸ existing sign-in ▸ gpt-5.6-sol
hermes chat -q "who are you"                   # the default agent answers

hermes profile create sage  --description "Customer support specialist. Answers tickets from the rules file, logs every ticket, escalates anything outside its authority to the manager."
hermes profile create atlas --description "Support manager. Reviews escalations, decides with the operator, hands approved actions back to the specialist."
hermes profile list                            # want: default, sage, atlas

cp ~/hermes-agent-ai-customer-support-team/profiles/sage/SOUL.md     ~/.hermes/profiles/sage/SOUL.md
cp ~/hermes-agent-ai-customer-support-team/profiles/atlas/SOUL.md    ~/.hermes/profiles/atlas/SOUL.md
cp ~/hermes-agent-ai-customer-support-team/profiles/sage/config.yaml  ~/.hermes/profiles/sage/config.yaml
cp ~/hermes-agent-ai-customer-support-team/profiles/atlas/config.yaml ~/.hermes/profiles/atlas/config.yaml

hermes -p sage chat -q "who are you"           # speaks as Sage
hermes -p atlas chat -q "who are you"          # speaks as Atlas
```

Building in order? Sage's SOUL names a skill that arrives in step 7 — remove the sentence `Follow your ticket-flow skill before replying to one.` now and put it back in step 7. Using an API key instead of a Codex seat: put `OPENROUTER_API_KEY=…` in `~/.hermes/profiles/<name>/.env` and set `provider: openrouter` + `model: <vendor>/<model>` in that profile's `config.yaml`.

## 3 · How the office talks

No commands. Agents hand work to each other on the server (Telegram does not deliver one bot's messages to another bot); agents reach people through the Telegram rooms; customers reach the desk by email.

## 4 · Telegram: one bot, one group, three rooms

In the Telegram app, not the terminal:

1. **@BotFather → `/newbot`** — display name `Marveno Support` (the company's name, not an agent's), username ending in `bot`. Copy the token when BotFather shows it.
2. **`/mybots` → the bot → Bot Settings → Group Privacy → Turn off** — before adding it to any group.
3. Create a group; add the bot as a plain member (admin not needed).
4. **Group Info → Edit → Topics → on.** This changes the group's id — read numbers only after this.
5. Create three topics: `support-inbox`, `escalations`, `desk-log`. Post one message in each.
6. Right-click each message → **Copy Link**: `t.me/c/1234567890/3/5` → group id is `-1001234567890`, room id is the **middle** number. Copy from one of the three rooms, not from General.

## 5 · Wire the front desk

```bash
vim ~/.hermes/.env
```

Add two lines (see `env.example`):

```
TELEGRAM_BOT_TOKEN=<BOT_TOKEN>
TELEGRAM_GROUP_ALLOWED_CHATS=<GROUP_ID>
```

```bash
grep -c '^TELEGRAM' ~/.hermes/.env             # want: 2

cd ~/hermes-agent-ai-customer-support-team
# your four numbers in place of the examples:
sed -i.bak 's/<GROUP_ID>/-1001234567890/g; s/<TOPIC_SUPPORT>/3/g; s/<TOPIC_ESCALATIONS>/4/g; s/<TOPIC_DESKLOG>/5/g' commands/escalate commands/ask-operator commands/resolve commands/resolve.telegram-only commands/tell-customer gateway-block.yaml && rm -f commands/*.bak gateway-block.yaml.bak
cat gateway-block.yaml >> ~/.hermes/config.yaml   # append — the installer already wrote this file

hermes gateway run --replace                   # foreground; give it its own terminal tab
```

First start downloads the Telegram library (a few seconds), prints a banner and two `[Telegram]` lines, then goes quiet. Quiet is success on v0.20.4 — the console prints failures only. Proof: type "hi" in **support-inbox**; the reply comes from Marveno Support, written by Sage.

```bash
hermes config get platforms                    # want: the telegram home_channel block echoed back
grep -i "telegram connected" ~/.hermes/logs/gateway.log   # the success line lives in the file log
```

Now the loophole: type `what's your refund policy?` in **desk-log**. That room has no route, so the default agent answers — an agent you have not briefed. Route each room: Ctrl-C in the gateway tab, then append the catch-all (it matches any Telegram conversation not named above; the specific routes still win) and start again:

```bash
cat >> ~/.hermes/config.yaml <<'EOF'
    - name: telegram-everything-else-to-sage
      platform: telegram
      profile: sage
EOF
hermes gateway run --replace
```

Ask the same question in desk-log again — this time Sage answers as the company's desk.

The gateway reads `.env` and `config.yaml` only at startup. After any change: Ctrl-C in the gateway tab, then `hermes gateway run --replace` again. On this foreground build do not use `hermes gateway restart` (it relocates the gateway into whatever shell you typed it in) or `journalctl --user -u hermes-gateway` (it reports on a service you never installed).

To keep the desk running after you log out, install it as a service instead:

```bash
hermes gateway install --start-on-login --start-now
hermes gateway status
journalctl --user -u hermes-gateway -f         # under the service, the logs live here
```

## 6 · The rules file

```bash
mkdir -p ~/marveno
cp ~/hermes-agent-ai-customer-support-team/rules.md ~/marveno/rules.md
touch ~/marveno/log.md                         # the agents write this file; you read it
cat ~/marveno/rules.md
```

Test from the inside — in **support-inbox**: `check ~/marveno/rules.md — a customer asks: can I import my old roasts from a spreadsheet?` Want: a no, matching the file's import line.

## 7 · Skills and the four commands

```bash
mkdir -p ~/.hermes/profiles/sage/skills/ticket-flow ~/.hermes/profiles/atlas/skills/escalation-review
cp ~/hermes-agent-ai-customer-support-team/profiles/sage/skills/ticket-flow/SKILL.md          ~/.hermes/profiles/sage/skills/ticket-flow/SKILL.md
cp ~/hermes-agent-ai-customer-support-team/profiles/atlas/skills/escalation-review/SKILL.md  ~/.hermes/profiles/atlas/skills/escalation-review/SKILL.md
hermes -p sage skills list                     # want: ticket-flow  local … enabled
hermes -p atlas skills list                    # want: escalation-review  local … enabled

mkdir -p ~/.local/bin
cp ~/hermes-agent-ai-customer-support-team/commands/escalate              ~/.local/bin/escalate
cp ~/hermes-agent-ai-customer-support-team/commands/ask-operator          ~/.local/bin/ask-operator
cp ~/hermes-agent-ai-customer-support-team/commands/resolve.telegram-only ~/.local/bin/resolve
cp ~/hermes-agent-ai-customer-support-team/commands/tell-customer         ~/.local/bin/tell-customer
chmod +x ~/.local/bin/escalate ~/.local/bin/ask-operator ~/.local/bin/resolve ~/.local/bin/tell-customer
grep -H -o 'chat_id=[^"]*\|message_thread_id=[^"]*' ~/.local/bin/{escalate,ask-operator,resolve,tell-customer}
```

Want from that last line: the same `chat_id` in each file; `escalate` and `resolve` → desk-log, `ask-operator` → escalations, `tell-customer` → support-inbox.

If you left the skill sentence out of Sage's SOUL in step 2, add it back now:

```bash
vim ~/.hermes/profiles/sage/SOUL.md
```

Then send `/new` in each routed room — a live session keeps the SOUL it started with; skills and `rules.md` are re-read every turn, the SOUL is not.

Proof of the loop — in **support-inbox**, you playing the customer: `Do you offer a student discount?` Want: "I've passed this to my manager." in support-inbox; 📤 then 📥 in desk-log; the manager's answer in support-inbox, delivered by Sage; three lines in the log:

```bash
cat ~/marveno/log.md                           # ESCALATED → RESOLVED → REPLIED
```

Optional — ask Hermes what it will ship to the model from a skill's description (it is capped at 60 characters, with no warning):

```bash
~/.hermes/hermes-agent/venv/bin/python - ~/.hermes/profiles/sage/skills/ticket-flow/SKILL.md <<'PY'
import sys
from agent.skill_utils import parse_frontmatter, extract_skill_description
fm, _ = parse_frontmatter(open(sys.argv[1]).read())
raw = str(fm.get("description", "") or "")
shipped = extract_skill_description(fm)
if not shipped:      print("FAIL: the model receives NOTHING - no description in the frontmatter")
elif shipped != raw: print(f"FAIL: {len(raw)} chars - the model only receives: {shipped!r}")
else:                print(f"OK: {len(raw)} chars - the model receives: {shipped!r}")
PY
```

## 8 · Email, both directions

Make the desk its own Gmail account (not a personal one) with 2-Step Verification on, then Google Account → Security → App Passwords → create one (the name is a label for you). You also need a second mailbox you control, to play the customer.

```bash
cat >> ~/.hermes/.env <<'EOF'
EMAIL_ADDRESS=<DESK_EMAIL>
EMAIL_IMAP_HOST=imap.gmail.com
EMAIL_SMTP_HOST=smtp.gmail.com
EMAIL_ALLOWED_USERS=<CUSTOMER_EMAIL>
EOF
vim ~/.hermes/.env                             # add: EMAIL_PASSWORD=<APP_PASSWORD>
grep -c '^EMAIL' ~/.hermes/.env                # want: 5
```

Add the email home address inside the existing `platforms:` block, level with `telegram:`:

```bash
vim ~/.hermes/config.yaml
```

```yaml
  email:
    home_channel:
      platform: email
      chat_id: <DESK_EMAIL>        # the desk's own address, not a customer's
      name: support-mailbox
```

```bash
grep -A4 '^  email:' ~/.hermes/config.yaml     # want: the email home_channel block with YOUR desk address
```

Restart the gateway (Ctrl-C, then):

```bash
hermes gateway run --replace                   # want: an [Email] line in the startup output
hermes config get platforms                    # want: both home_channel blocks
```

Proof: from the customer mailbox, email the desk `when do orders ship?`. Within about a minute the reply lands in the **customer's** inbox, threaded under the same subject. Leave the desk's own mailbox unopened — the desk finds work by asking for unread mail, and a message you open is one it is not offered.

The second door — replies into an existing thread after a decision:

```bash
curl -sSL https://raw.githubusercontent.com/pimalaya/himalaya/master/install.sh | PREFIX=~/.local sh
himalaya --version                             # want: a v2.x line
himalaya                                       # the wizard: Y → the desk address → IMAP + SMTP imap.gmail.com → PLAIN → store raw → the App Password
# if the wizard fails: cp himalaya/config.example.toml ~/.config/himalaya/config.toml and replace its placeholders
himalaya account list                          # the wizard names the account after the provider: gmail
vim ~/.config/himalaya/config.toml             # rename [accounts.gmail] → [accounts.support]
himalaya account check --account support       # want: imap: OK / smtp: OK
himalaya envelope list --account support       # the ids in the first column are what a reply uses

cp ~/hermes-agent-ai-customer-support-team/commands/tell-customer-email ~/.local/bin/tell-customer-email
cp ~/hermes-agent-ai-customer-support-team/commands/resolve             ~/.local/bin/resolve      # the finished resolve: checks the mailbox, picks the door
chmod +x ~/.local/bin/tell-customer-email ~/.local/bin/resolve
grep -c himalaya ~/.local/bin/resolve          # want: 1
```

## 9 · Run the desk

From the customer mailbox, one at a time, watching desk-log and escalations in Telegram:

1. `How do I export my roast history?` — answered from the rules; one REPLIED line.
2. `We're a three-person roastery — is there a team plan where we can all log to one account? Even a rough sense of when would help us plan.` — the manager decides; ESCALATED → RESOLVED → REPLIED; no date promised.
3. `I sold my roaster back in January and stopped roasting, but I completely forgot to cancel — just noticed I've been charged $9 every month since. My mistake entirely — any chance of refunding those months? About $63, I think.` — a 🔺 card in escalations; reply there in words; the answer lands in the customer's thread; PENDING OPERATOR → RESOLVED → REPLIED.

```bash
tail -5 ~/marveno/log.md
```

The routine — work on a clock, owned by the front desk (on v0.20.4 an agent-owned job cannot deliver under this one-token design; it blocks without a message):

```bash
hermes cron create "every 15m" "SCHEDULED ROUTINE, not a customer ticket — do not escalate. Read ~/marveno/log.md and post a short shift summary: how many tickets you have handled, and name any that were escalated or declined. Four lines maximum. If the log has no rows yet, reply exactly: NO TICKETS YET." \
  --name "support-shift-summary" --deliver "telegram:<GROUP_ID>:<TOPIC_DESKLOG>" --workdir ~/marveno
hermes cron list                               # the job, its Next run, and later its Last run + any error
hermes cron runs                               # durable run history: proof it FIRED
hermes cron run support-shift-summary          # fire it now instead of waiting: Ran now: succeeded.
```

`every 15m` repeats; a plain `15m` runs once. The tick lands up to a minute after `Next run:`. Reports go to desk-log, the watch room — not the customer counter.

## Before real customers

Rotate the credentials you used while building (bot token via @BotFather `/revoke`, the App Password in Google Account, the model seat), and read the article's *Where to take it from here* before opening `EMAIL_ALLOW_ALL_USERS=true`.
