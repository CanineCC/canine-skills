---
description: Fix what the scan found in this repository, most important first, verified by the next scan
---
Use the **cca** MCP server to work this repository's findings.
Your agent knows it as **cca**, or by the name you gave the server when you added it.

This is /cca-fix version 5.1.0. `next_task`'s `progress` block (and `audit`'s `queue` block) states the newest version as
`fixCommandVersion`. If that is newer than this file's, tell the person once, in one sentence in their language, that a newer
/cca-fix exists and how to get it (in Claude Code: `/plugin marketplace update canine`, or switch on auto-update under `/plugin`,
Marketplaces), then carry on with the work. Never update it yourself.

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
   - **Fix it**: run the project's tests before you change anything, change the code at every instance, then run the tests
     again. A test that passed before and fails now means the fix is not done: repair it or undo it. Then commit the change the
     way this repository is worked (see **Check every fix with the scan**) and record `set_cause_disposition(dimension, causeKey,
     "FixedAwaitingScan")`, which takes the cause off `next_task` while it waits for its check. Never claim a fix you didn't
     make or a test run you didn't do.
   - **Dispute it**: `dispute_finding` only if the finding is wrong: what it describes is not there, or the scanner read code that
     is not the product's (a fixture, generated or vendored code, a developer script that never ships). Quote the code at the cited
     line as `evidence`, and say how the scanner could have known. Pass further instances of the same root cause as
     `siblingFingerprints`: one dispute, not many. Disputes are capped at 60 calls per hour per key, and a grouped dispute counts as
     one call. A maintainer decides: an upheld dispute corrects the scanner for everyone, a declined one comes back with the reason.
   - **Accept it**: `set_cause_disposition(dimension, cause, "Accepted", note)` if the finding is correct and you are deliberately
     not acting (a design decision, a trade-off). Say in the note why. The owner confirms it on the findings page. Where an
     acceptance is refused because it costs the owner, fix it, or leave it open and say so: do not dispute a correct finding.
3. Call `next_task` again, every time. Stop only when it returns nothing.

**Check every fix with the scan.** The scan is the judge of a fix, not your tests and not your own reading of the code. Work the
way this repository is already worked. Tell which from what you can see: merge commits or pull requests in `git log`, a
`CONTRIBUTING` file or the person's own instructions, or a push to the default branch that is refused. If you cannot tell, ask
the person once.

*With branches or pull requests:* check before you resolve.
- Work on one branch for the whole session, made from the default branch (for example `cca-fix-` and today's date), and commit
  each cause's fixes to it.
- After a few causes, push the branch, call `request_rescan` with `branch`, then read `get_branch_preview` until it is `Done`,
  no more than once a minute. Branch previews have a limit of their own per repository, apart from full scans, so check several
  causes in one preview, never one preview per cause.
- When it is `Done`, a cause is verified when its findings are in `leaving` and nothing is in `arriving`. Then `resolve_finding`
  each of its findings, with a short note: what you changed, that the tests still pass (or that the project has none), and
  that the branch preview showed the finding leave. A finding that did not leave is not fixed: put its cause back with
  `set_cause_disposition(dimension, causeKey, "NeedsDecision")` and work it again. A finding that arrives came with your change:
  fix it on the same branch, in the same session, before you go on. (One that arrives in code you did not touch can come from
  commits the default branch's latest scan has not seen yet: say so rather than change other people's work.)
- Never resolve on your own word when a preview was possible. If it was not (you cannot push to the remote, or `request_rescan`
  answers `Disabled`, `BranchNotResolved` or `Unavailable`, or `RateLimited` with a `retryAfter` later than this session can
  wait), say so and why, and check instead with the tests and with `check_dependencies` on the diff of the session
  (`git diff -U100000` against the commit the branch started from; it judges C#, Java, Kotlin and Scala). Resolve with a note
  saying which check you used.

*Straight to the default branch:* the next scan is the check, so resolve nothing on your own word.
- Commit each cause's fixes and push them to the default branch as the person does. Before each push, run `check_dependencies`
  on the diff (C#, Java, Kotlin and Scala) and fix what it reports.
- When the list is done, call `request_rescan` once (no `branch`), then read `audit` no more than once a minute until its scan
  names the commit you pushed last. A finding that the scan no longer finds is gone from the list and the score: that is its
  check, and it needs nothing more from you. A cause whose findings are still there is not fixed: put it back with
  `set_cause_disposition(dimension, causeKey, "NeedsDecision")` and work it again. A new finding in code you changed came with
  your change: fix it the same way.
- If `request_rescan` is refused, say so and why: the fixes are checked by the project's next scheduled scan.

Never write your own checker that copies the scan's limits (a script with its own thresholds or counts): it drifts from the
measurement and passes code the scan will fail. The scan is the check.

**Done** means `next_task` returned nothing and one last check of everything this session changed, a branch preview or the
rescan of the default branch, shows nothing arriving; or you say why it could not run and what you checked instead.

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
- If you run low on context before the list is done, write down the goal, the `progress` block, what you closed, how this
  repository is worked (and the branch, if any) and which causes still wait for their check, so the next session carries on with
  the whole job.

When `next_task` returns nothing, report to me in three piles, in plain words: what you **fixed** and why it matters (the next scan
checks it); the **claimed scanner errors** you disputed and why (they await a maintainer's answer, and they are claims, not
verdicts); and what you **accepted** or left open and why (as a list I can confirm on the findings page). Then:
- **Checked**: how the work was checked and what it showed. With a branch: the last preview's commit, and that nothing arrives
  (or what arrived and what you did about it). Straight to the default branch: the rescan's commit, which causes it confirmed
  gone and which went back, or, if no rescan could run, that the next scheduled scan checks them. If no check could run, say why
  and what you checked instead.
- **Rules to keep**: the few rules the code broke most often in this session, from the causes you worked, most places first,
  each in one plain sentence on how to write new code that keeps it, so the team and its AI avoid them next time.
- **The branch** (when you worked on one): its name, and that the fixes reach the scan only once it is merged into the default
  branch. A resolved finding still in the default branch at its next scan is reported back as a fix that did not hold, so offer
  to merge it now (or to open a pull request, if that is how this project merges). Once it is merged, call `request_rescan` if
  the repository allows it; otherwise tell me the next scan checks the fixes.

$ARGUMENTS
