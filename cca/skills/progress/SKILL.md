---
name: progress
description: "Reports how the code's quality has moved over time and whether fixes held, apart from changes in how it is measured. Use whenever someone asks whether the code got better or worse, how quality is trending, what changed since a date, or whether the fixes their AI made actually worked. Uses the cca MCP server's tools."
---

# Quality progress

## Words
- **server**: the cca MCP server; its tools are named `cca:<tool>`. A **scan** is one run of it over the repository, named by
  its commit and date. A **finding** is one problem the scan found at a place in the code.
- **dimension**: one kind of check, with an id such as `D17`. A **lens** groups dimensions; the **headline** is the overall score.
- **plan**: the ranked list of causes in `cca:audit` (a **cause** groups findings with one remedy); each cause is a **plan row**.
- **fix**: a resolution the person or their AI reported. `queue.resolutionsImproved` holds the fixes that held (their dimension
  scored higher), `queue.resolutionsRefused` the fixes the scan found again.
- **trend point**: one scan on the trend. A **move** is the difference between two trend points' scores. A **mark** is a point
  on the trend where the way of measuring changed; a move across a mark is partly or wholly a change in measuring.
- **unexplained move**: two neighbouring trend points whose headline differs with no commit and no mark between them (an empty
  `git log <commit a>..<commit b>`).
- **summary**: the three sentences at the top of the report.
- **What the code did** over the period: `codeMove` from `cca:get_trend` when the period is the whole trend; for a shorter
  period, the `headline` at its last trend point minus the `headline` at its first, minus each mark's `headlineDelta` inside it,
  minus each unexplained move.
- **A lens's move** over the period: its score at the last trend point minus its score at the first, raw.

## Rules for this skill
- **Signing in.** If the server's tools are missing or say you must sign in, stop before you write anything. Tell the person in
  one line: in Claude Code, type /mcp, choose cca and sign in (a server added during this chat may need a new chat). Write
  without the scan only if they ask, and then say at the top that the scan was not used.
- **Defaults.** Reader: someone who does not write code (rule below). Focus: the whole repository. Period: every trend point
  `cca:get_trend` returns; "the last sprint" is the last two weeks unless a tag or date says otherwise. Form: a reply in the
  chat. The person's own words win over these defaults: the reader, the focus, the period and the form (a PDF or a document:
  make the file with what your tool has, or say it cannot). When something is missing, take the default, write the report,
  and say in one line which default you used. Never hand the person a list of technical options to choose from.
- **Scan and reading.** Keep apart what the scan measured (name the scan) and what you read (name the commits or files). A link
  between a commit and a move is your reading: say so.
- **Numbers.** Every number comes from the server's answer or a count you made with git; a number about the code now names its
  scan, a trend point names its date. Never present a mark's move or an unexplained move as the code's, and compare two scans
  only along the trend.
- **A reader who does not write code.** The summary first, on one screen, with the same numbers as the body. No tool
  names, commit hashes or dimension ids; a lens or dimension only by what it measures, in a few words.

## Workflow

Copy this checklist and tick it off:

```
Progress report:
- [ ] 0. Server answers (Signing in)
- [ ] 1. What the code did
- [ ] 2. Fixes that held and fixes that did not
- [ ] 3. The three lenses that moved most
- [ ] 4. The reason for each move
- [ ] 5. Write the report
- [ ] 6. Check the report and add the closing paragraph
```

**0. Server answers (Signing in).** Call `cca:get_trend` (no arguments). If the tool is missing or asks you to sign in: follow
the sign-in rule. If it returns no trend point: write "There is no scan of this repository yet." and stop. If the period holds
one trend point: the report is its date and `headline`, the sentence "A move needs two scans.", and step 2's two fix sections;
skip steps 1, 3 and 4 and every step 6 check about moves and lenses.

**1. What the code did.** Keep the trend points inside the period (each has a date, `headline`, a commit and its lens scores) and
the `marks` inside it (each with `at`, its `sentence` and `headlineDelta`). File each unexplained move with
`cca:report_platform_issue`, `area` "scan-pipeline", `title` "Unexplained trend move" and a `description` with both dates,
commits and headlines; if `cca:report_platform_issue` fails, say so under "Changes in how it is measured". If `git log`
cannot read both commits (no clone here, or a commit missing from it): count no move as unexplained and say so in one line
under "Changes in how it is measured". Then work out what the code did.

**2. Fixes that held and fixes that did not.** Call `cca:audit` (no arguments). Keep `scan` (its date), the `plan` rows in order
(each with its dimension, lens and current /10), `queue.resolutionsImproved` and
`queue.resolutionsRefused` (each refused fix with its `refusedResolution` note). For each fix,
keep its dimension and the file its finding names.
If `cca:audit` fails: both fix sections hold the one line "The fixes could not be read from the scan."; leave out the
summary's third sentence (the one thing to do next), step 3's `cca:get_dimension` calls and the step 6 fix check.

**3. The three lenses that moved most.** Keep the three lenses whose move is largest by size, up or down. On a tie, keep
the lens with the larger move between two neighbouring trend points; if fewer than three lenses moved, keep those that did; if
none moved, write "No lens moved over the period." under both "What improved" and "What got worse". For each kept lens that
fell, call `cca:get_dimension` once for each plan row naming that lens, with `dimension` set to the row's dimension; keep its
number of findings, the files they name, and `reliability.measured`. If no plan row names a fallen lens: write that it has no
open cause.

**4. The reason for each move.** For each kept lens, take the two neighbouring trend points between which it moved most and run
`git log --stat <commit a>..<commit b>`; if it fails, the reason is "not read: the commits are not in this checkout". For a
lens that fell, read the commits touching the files from step 3; for one that rose, the commits whose files bear on what it
measures. Say the link is your reading; if a mark lies between those two trend points,
say its sentence beside the reason. For each refused fix, open its file and say what is still there.

**5. Write the report** in this shape. Replace an empty section with one line ("Every reported fix held."):

```markdown
# How the code has moved, <first date> to <last date>

<Summary: what the code did, with both dates; what changed in how it is measured; the one thing to do next, the first
plan row in a few words.>

## What improved
<Each kept lens that rose: what it measures in a few words, its two scores with their dates, the reason from step 4.>

## What got worse
<Each kept lens that fell: the same, then each of its dimensions by what it measures, with its number of findings and current
/10 from the scan of <date>. Present a dimension whose reliability.measured is false as neither certain nor noise.>

## Fixes that held
<Each from resolutionsImproved: what the fix changed and where, as the scan of <date> found.>

## Fixes that did not hold
<Each from resolutionsRefused: what is still there, from its file and its refusedResolution note.>

## Changes in how it is measured
<Each mark's own sentence with its date. Each unexplained move: its two dates and size, said plainly as not the code's.
Unless the period is the whole trend: one line that lens moves include any change in measuring.>
```

**6. Check the report and add the closing paragraph.** Check, one by one:
- what the code did follows its definition, and no mark's move or unexplained move is called the code's;
- every unexplained move was filed, or the report says the call failed;
- every fix from step 2 is under "Fixes that held" or "Fixes that did not hold";
- only the three lenses of step 3 appear under "What improved" and "What got worse", and unless the period is the whole
  trend the report says once that lens moves include any change in measuring;
- every number names its scan or trend date, and every reason names what you read;
- for this reader: no tool names, commit hashes or dimension ids, and each lens is said by what it measures.

Correct each item that fails, then check the whole list again. When all pass, add the closing paragraph: a short paragraph on what
the server gave this report, only from the `contribution` items that changed it, each said as what the report would otherwise
have got wrong or lacked. If none changed it, leave it out.

## Examples

For the default reader, a line under "What got worse":

> Tests, the share of the code that automatic tests run, fell from 71 to 64 between the scans of 3 and 10 November. In that
> week the new export feature added about 900 lines with no tests (my reading of the changes). Untested code now shows 12
> findings and scores 6.4 out of 10 in the scan of 10 November.

For a developer who asked for detail, the same line:

> The Tests lens fell from 71 to 64 between the scans of 3 November (`a1b2c3d`) and 10 November (`b7c8d9e`). `git log --stat`
> over that range shows `src/export/` gaining about 900 lines and no test files; linking the export commits to the fall is my
> reading. D17 holds 12 findings at 6.4/10 in the scan of 10 November.
