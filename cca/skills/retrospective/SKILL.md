---
name: retrospective
description: "Writes a retrospective of the code's quality over a sprint or period: what went well, what did not, which fixes held, and three things to try next. Use whenever someone asks for a retro, a retrospective, a sprint review or 'how did we do' on the code. Uses the cca MCP server's tools."
---

# Retrospective

## Words
- **server**: the cca MCP server; its tools are named `cca:<tool>`. A **scan** is one run of it over the repository, named by
  its commit and date. A **finding** is one problem the scan found at a place in the code.
- **dimension**: one kind of check, with an id such as `D15`. A **lens** groups dimensions; the **headline** is the overall score.
  The **security dimensions** are `D13` secrets in the code, `D28` secrets in the git history, `D29` security bugs found by
  static analysis, `D30` dependencies with known vulnerabilities, `D31` container and infrastructure configuration, `D32`
  personal data.
- **plan**: the ranked list of causes in `cca:audit` (a **cause** groups findings with one remedy).
- **reported fix**: a finding someone reported done; it **held** when its dimension scored higher, and **did not hold** when a
  later scan found it again.
- **trend point**: one scan on the trend. A **move** is the difference between two trend points' scores. A **mark** is a point
  on the trend where the way of measuring changed; a move across a mark is partly or wholly a change in measuring.
- **unexplained move**: two neighbouring trend points whose headline differs with no commit and no mark between them (an empty
  `git log <commit a>..<commit b>`).
- **What the code did** over the period: `codeMove` from `cca:get_trend` when the period is the whole trend; for a shorter
  period, the `headline` at its last trend point minus the `headline` at its first, minus each mark's `headlineDelta` inside it,
  minus each unexplained move.
- **A lens's move** over the period: its score at the last trend point minus its score at the first, raw.

## Rules for this skill
- **Signing in.** If the server's tools are missing or say you must sign in, stop before you write anything. Tell the person in
  one line: in Claude Code, type /mcp, choose cca and sign in (a server added during this chat may need a new chat). Write
  without the server only if they ask, and then say at the top that no scan was used.
- **Defaults.** Reader: the team, at its meeting, who write code; a reader who does not write code (a manager, a client) gets
  the same retrospective by the rule for that reader below. Period: the last sprint, the last two weeks unless a tag or date
  says otherwise. Form: a reply in the chat. The person's own words win over these defaults: the reader, the focus, the
  period and the form (a PDF or a document: make the file with what your tool has, or say it cannot). When something is
  missing, take the default, write the retrospective, and say in one line which default you used.
  Never hand the person a list of technical options to choose from.
- **Scan and reading.** Keep apart what the scan measured (name the scan) and what you read (name the commits or files). A link
  between a finding and a commit is your reading: say so. A correct finding is never a matter of taste.
  Never explain how the score is counted.
- **Numbers.** Every number comes from the server's answer or a count you made with git; a number about the code now names its
  scan, a trend point names its date. Never present a mark's move or an unexplained move as the code's.
- **Order of work.** Open security findings first, then the plan rows from the top.
- **A reader who does not write code.** The short version first, on one screen, with the same numbers as the body. No tool
  names, commit hashes or dimension ids; each scan by its date only; a lens or dimension only by what it measures.

## Workflow

Copy this checklist and tick it off:

```
Retrospective progress:
- [ ] 0. Server answers (Signing in)
- [ ] 1. What the code did
- [ ] 2. Reported fixes that held and that did not
- [ ] 3. What the scan holds now
- [ ] 4. What was worked on
- [ ] 5. Write the retrospective
- [ ] 6. Check the retrospective and add the closing paragraph
```

**0. Server answers (Signing in).** Call `cca:get_trend` (no arguments). If the tool is missing or asks you to sign in: follow
the sign-in rule. If it returns no trend point: write "There is no scan of this repository yet." and stop. If the period holds
fewer than two trend points, open with "The scan shows no move for this period."

**1. What the code did.** Skip this step if the period holds fewer than two trend points. Keep the trend points inside the
period (each has a date, `headline`, a commit and its lens scores) and the `marks` inside it (each with `at`, its `sentence`
and `headlineDelta`). Report each unexplained move with `cca:report_platform_issue`, `area` "scan-pipeline", `title`
"Unexplained trend move" and a `description` with both dates, commits and headlines. Then work out what the code did. If
`cca:report_platform_issue` fails: say so under "Changes in how it is measured".

**2. Reported fixes that held and that did not.** Call `cca:audit` (no arguments). Keep `queue.resolutionsImproved` (reported
fixes that held) and `queue.resolutionsRefused` (reported fixes that did not hold, each with its `refusedResolution` note).
For each reported fix, keep its dimension and the file its finding names.
If `cca:audit` fails: skip step 2, the plan parts of step 3 and the step 6 check on reported fixes; base the three things to
try on security findings only, and say in one line that the plan could not be read.

**3. What the scan holds now.** From the same `cca:audit` answer keep `scan`, `queue.causesRemaining`, `queue.pointsRemaining`
and the `plan` rows in order. Call `cca:list_findings` once for each security dimension, with that dimension and `state`
"open", and read `totalMatching`. Keep every lens's move; the largest by size is the lens that moved most (with fewer than two
trend points: leave out every lens move and use the lens of the first plan row instead). For each plan row
naming that lens, call `cca:get_dimension` with `dimension` set to the row's dimension; keep its number of findings, the files
they name, and `reliability.measured`. If no plan row names it: write that it has no open cause.

**4. What was worked on.** Skip this step if the period holds fewer than two trend points. Run `git log --stat <first trend
point's commit>..<last trend point's commit>` and group the work by top-level folder. Read the commits that touched the files
from step 3 and each file of a reported fix that did not hold. If `git log` fails (no local clone, a shallow clone, a commit
not fetched): say in one line that the commits could not be read and leave commit references out.

**5. Write the retrospective** in this shape:

```markdown
# Retrospective, <first date> to <last date>

<One sentence: what the code did, with its dates, and the one thing that stands out.>

## What went well
<Each lens that rose (its two scores and dates) and each reported fix that held (its file), with the commits behind it.>

## What did not go well
<Each reported fix that did not hold (its file, its refusedResolution note, the commit that touched it) and each lens that fell.>

## Where the code stands now
<From the scan of <date>: open security findings by dimension first, then causes remaining and points remaining, then the
lens that moved most and each of its dimensions with its number of findings. Present a dimension whose reliability.measured
is false as neither certain nor noise.>

## Changes in how it is measured
<Each mark's own sentence with its date, and each unexplained move, apart from the code. Unless the period is the whole
trend: one line that lens moves include any change in measuring.>

## Three things to try next
1. <In the order of work. Each checkable by a later scan.>
2. ...
3. ...
```

**6. Check the retrospective and add the closing paragraph.** Check, one by one:
- with two or more trend points: what the code did follows its definition, and every unexplained move was reported or the
  failed call is said;
- every item under "What went well" and "What did not go well" names its scan, and its commit for the team unless step 4
  said the commits could not be read;
- every reported fix from step 2 appears under one of those two sections;
- every link between a finding and a commit is marked as your reading;
- every number names its scan or trend date;
- the three things to try follow the order of work, and each can be checked by a later scan;
- unless the period is the whole trend, the retrospective says once that lens moves include any change in measuring;
- for a reader who does not write code: no tool names, commit hashes or dimension ids;
- the retrospective says what happened to the code, and names or blames no person.

Fix each item that fails, then check the whole list again. When all pass, end with the closing paragraph the server's
instructions describe, from the `contribution` items that changed this retrospective.

## Examples

For the team, the opening and one item under "What did not go well":

> Headline 68 at the scan of 30 October (`3f9a1c2`), 73 at the scan of 13 November (`8d0e4b7`); one mark inside the sprint (a
> new way of counting duplicated code) moved it +2, so the code did +3.
>
> The reported fix to `src/auth/session.ts` did not hold: the scan of 13 November finds the token still logged ("log call
> kept at line 88"). Commit `5c21e0a` touched the file but moved the call rather than removing it (my reading).

For the team, a dimension whose `reliability.measured` is false, under "Where the code stands now":

> `D17`: 12 findings in `src/legacy/`; this check is not yet measured as reliable, so treat them as worth a look, not as settled.

For a manager who does not write code, the same sprint:

> Between the scans of 30 October and 13 November the code itself got 3 better on its quality score, out of a rise of 5; the
> other 2 came from a change in how copied code is counted. One reported fix, to how sign-in sessions are logged, did not
> hold, and the team will finish it next.
