# /hush

A Claude skill that makes Claude stop narrating and just execute.

---

## The problem

Claude narrates everything by default.

Before touching a file:
> "I'll start by exploring the codebase structure."

Between tool calls:
> "Now applying the fixes. Let me check the payment service next."

After finishing:
> [500 word bug report with headers, sub-bullets, and unsolicited recommendations]

Every word Claude writes enters the context window. Claude then re-reads its own narration on every subsequent turn — you pay input token cost for it twice. On long sessions it compounds fast.

---

## What /hush does

- First output is always a tool call, never text
- Zero text between tool calls
- No phase announcements, no self-narration, no mid-task commentary
- One ultra-compressed summary at the end — `files:` / `changes:` / `flagged:` sections, 20 lines max
- Inline code comments capped at one line, no TODOs left behind

Tool call indicators (bash, file reads, MCP) still show. Those are action visibility, not narration.

---

## Output before and after

**Without /hush**

```
I'll start by exploring the codebase structure.
[tool call]
Small codebase — let me read all of it.
[tool call]
Now I have the full picture. Fixing the data layer first.
[tool call]
Now the inventory service — closing the TOCTOU gap.
[tool call]
Now the payment service — atomic wallet debit.
[tool call]
Now the order service — core sequencing fix.
...
[500 word summary with headers]
```

**With /hush**

```
[tool call]
[tool call]
[tool call]
[tool call]
[tool call]
...

task complete

files:
- config/db.js: fixed
- services/inventoryService.js: fixed
- services/paymentService.js: fixed
- services/orderService.js: fixed

changes:
- db.js line 37: atomic debitBalance added, stops concurrent wallet overdraw
- db.js line 84: atomic reserveStockIfAvailable closes TOCTOU gap
- inventoryService.js line 29: partial reservations released on any item failure
- paymentService.js line 13: wallet debit atomic, credits back if record write fails
- orderService.js line 61: order row created before charge, payment always has an order
- orderService.js line 157: status validated against VALID_TRANSITIONS

flagged:
- services/authService.js: hardcoded JWT secret, outside scope
```

---

## Results

Testing done on Opus 5, High thinking, same codebase and prompt both runs.

| | Without /hush | With /hush |
|---|---|---|
| Message tokens | 87.7k | 70k |
| Total context | 125.7k | 108k |

Results vary by model, task, and session length. Token savings from suppressed narration are consistent — if Claude produces zero text between tool calls it uses fewer message tokens than a Claude that writes paragraphs between every tool call. That part is math, not a claim.

---

## Best setup

**Opus + High thinking + /hush**

Claude reasons in two layers. The thinking block (invisible scratchpad) is where the real reasoning happens. The message layer is where Claude re-narrates that reasoning out loud. Extended thinking handles the IQ. /hush eliminates the redundant re-narration layer.

Without extended thinking, suppressing narration removes some reasoning capacity. With extended thinking, that reasoning already happened in the scratchpad before a single tool ran. /hush then strips the redundant layer cleanly.

---

## Install

**Claude Desktop / Claude.ai**

```
~/.claude/skills/hush/SKILL.md
```

Copy `SKILL.md` from this repo into that path. Create the folder if it doesn't exist.

Then type `/hush` before any task to activate it.

**Claude Code (terminal)**

Add the contents of `SKILL.md` to your `CLAUDE.md` file in your project root.

---

## When to use it

Use /hush for mechanical tasks where Claude knowing the plan matters less than executing it fast — refactors, migrations, bug fixes with clear symptoms, file operations, boilerplate generation.

Skip /hush for open-ended debugging or security audits where Claude thinking out loud is doing real work — in those cases the narration isn't waste, it's reasoning.

---

## Made by LoomingAI

