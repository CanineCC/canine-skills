---
description: Fix what the scan found in this repository, most important first, verified by the next scan
---
Use the **cca** MCP server to work this repository's findings.
Your agent knows it as **cca**, or by the name you gave the server when you added it.

**The goal: every finding the scan reported is looked at and given the right ending: fixed, disputed or accepted.** `next_task`
serves one **cause** at a time (the problem a group of findings share), with every place it occurs. The server decides the order:
security findings first, then the rest by what fixing them is worth. That order decides what you work on first. It never decides
how much of the list you work, and it never decides WHAT you change.

**Fix the real cause, always.** Before you edit, describe the problem in words an engineer who has never seen the scanner would
recognise. If the only description is "it is flagged", the code may be fine: dispute or accept, don't change it to silence it.

**Every rule is measured the same way on every repository.** The score moves only when the code changes, or when a maintainer
agrees the scanner was wrong and corrects it for everyone. Never change code only to move the score (a rename, a baseline that
freezes the defect, a list of exceptions). Never call a correct finding a matter of taste: it is either wrong, or it is correct and
you fix it or accept it.

Loop:
1. Call `next_task`. It gives you the next cause, fully briefed: what to do, every instance (file:line and fingerprint), the
   `causeKey` and a `progress` block. A packet carries at most 50 instances; when a cause has more, work them and call again.
2. Give the cause **one of three endings**:
   - **Fix it**: change the code at every instance, then `resolve_finding` each with a short note saying what you changed. The
     next scan checks it, so never claim a fix you didn't make.
   - **Dispute it**: `dispute_finding` only if the finding is wrong: what it describes is not there, or the scanner read code that
     is not the product's (a fixture, generated or vendored code, a developer script that never ships). Quote the code at the cited
     line as `evidence`, and say how the scanner could have known. Pass further instances of the same root cause as
     `siblingFingerprints`: one dispute, not many. Disputes are capped at 60 calls per hour per key, and a grouped dispute counts as
     one call. A maintainer decides: an upheld dispute corrects the scanner for everyone, a declined one comes back with the reason.
   - **Accept it**: `set_cause_disposition(dimension, cause, "Accepted", note)` if the finding is correct and you are deliberately
     not acting (a design decision, a trade-off). Say in the note why. The owner confirms it on the findings page. Where an
     acceptance is refused because it costs the owner, fix it, or leave it open and say so: do not dispute a correct finding.
3. Call `next_task` again, every time. Stop only when it returns nothing.

Rules:
- **Count in items, as the app does.** `audit` has a `list` block: the items on the list the product's app shows, in total and per
  area, and every `progress` block carries `itemsOnList`. One item can be several findings (one package's licence parts, one function's
  two measures, one weakness two checks found at one place); a packet's `sameItem` names the other findings of an instance's item: fix
  it once and resolve each of them. When you report numbers to the person, use these, so they match what the app shows.
- **Only an empty `next_task` ends the loop.** Don't stop because the rest looks less important.
- **Stall guard:** if `next_task` hands you the same cause twice with nothing resolved or decided in between, stop and say so.
- **Advice** (`kind: "advice"`) comes after the findings, one at a time: the Recommendations list the person sees in the app. It has
  no instances and never moves the score. Fix it or hide it: make the change if it applies, then `set_cause_disposition(dimension,
  causeKey, "FixedAwaitingScan")` (the next scan checks it); if it does not fit this project, `set_cause_disposition(dimension,
  causeKey, "Accepted", note)` hides it on this project for good, in the app too. Report each hide with its reason.
- An **advisory** finding (`advisory: true`) does not move the score and cannot be disputed: fix it, accept it, or use
  `flag_advisory` with a note if it looks wrong.
- Suppressing a finding **in the code** (an inline suppression, an ignore entry, a config exemption) is a human decision, never yours.
- If you run low on context before the list is done, write down the goal, the `progress` block and what you closed, so the next
  session carries on with the whole job.

When `next_task` returns nothing, report to me in three piles, in plain words: what you **fixed** and why it matters (the next scan
checks it); the **claimed scanner errors** you disputed and why (they await a maintainer's answer, and they are claims, not
verdicts); and what you **accepted** or left open and why (as a list I can confirm on the findings page). If the repository allows it, call `request_rescan`; otherwise tell
me the next scan checks the fixes.

$ARGUMENTS
