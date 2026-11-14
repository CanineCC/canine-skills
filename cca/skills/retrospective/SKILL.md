---
name: retrospective
description: "Writes a retrospective of a sprint or period for the team's meeting: what was done, by theme, the bugs found along the way, what went well and what did not in how the work was done, where the code stands, and concrete things to try next sprint, each backed by the commits. Use whenever someone asks for a retro, a retrospective, a sprint review or 'how did we do' on the code. Uses the cca MCP server's tools for the few facts git cannot give: what the code did to the score apart from changes in measuring, and fixes that did not hold."
---

# Retrospective

The reader is the team at its retro. Afterwards they can say what the period went into, which habits to keep and which to
change, and leave with tries to act on next sprint. Git and the team's rules carry it. The server adds what git cannot: how
much of a score move was the code and how much a change in measuring, fixes that did not hold, areas not measured.

## Words
- **server**: the cca MCP server, tools named `cca:<tool>`. A **scan** is one measurement of the repository by the server,
  named by its date and commit. An **item** is an issue or backlog item a commit names (`#212`).
- **score**: one number per scan, 0 to 100: `headline` on `cca:get_trend`, the same number as `health` on `cca:audit`.
- **trend point**: one scan on `cca:get_trend`. Its **trend lenses** give a **lens value** per area (security,
  accessibility, code health...; say each by its area). A **mark** is a change in the way of measuring, with its `sentence`
  and `headlineDelta`.
- **unexplained move**: neighbouring trend points whose score differs with no commit and no mark between them.
- **security dimension**: one measure under the security lens, 0 to 10, passed to the tools as `dimension`: `D13` secrets in
  the code, `D28` secrets in git history, `D29` static-analysis security bugs, `D30` vulnerable dependencies, `D31` container
  and infrastructure settings, `D32` personal data. The **audit lens entries** are the entries under `lenses` on `cca:audit`.
- **fix that did not hold**: a finding the team marked fixed that the next scan still found (`queue.resolutionsRefused`).
- **the team's rules**: what the repository's own docs ask of the work (step 1). The skill's own rules are under Rules.

## Rules
- **Signing in.** If the server's tools are missing or say you must sign in, stop before you write anything. Tell the person in
  one line: in Claude Code, type /mcp, choose cca and sign in (a server added during this chat may need a new chat). Write
  from git alone only if they ask, and then say at the top that no scan was used.
- **Defaults.** The period is the last two weeks up to today by calendar date; the reader writes code; the form is Markdown in
  the chat, 80 to 120 lines. The person's own words win: the period (a tag, a sprint, dates), the reader, the focus ("what
  should we try" puts the tries first) and the form (a PDF or a document: make it with what your tool has, or say it cannot).
  Take the default for what is missing. Never hand the person a list of technical options to choose from.
- **Credit a score move only to commits the scan had read**: commits in `git log <A>..<B>` for the trend points around it that
  touch that area. A commit after B is not behind B's score, however well it fits. A move across a mark is measuring; an
  unexplained move is never the code. Word a link as a reading ("likely from").
- **Every count is exact**: count non-merge commits and merges apart, recount each with its command before you write it, and
  take lines from the net diff. Write only files, commit ids, items and quotes you read in the repository.
- **The team's rules are the yardstick**: name the rule you hold the work against. When a doc and the code or a newer doc
  disagree, say which is right.
- **What happened, not who, to the team.** Name no person and blame no one; write no "I", "my count", "I didn't check" (what
  cannot be checked is an open question). The score is one section, not the spine; leave out scan internals the team cannot
  act on. A correct finding is never a matter of taste.
- **A reader who does not write code**: put a one-screen short version first with the same numbers; use no tool names, commit
  ids, dimension ids or file paths; name each scan by its date.

## Workflow
Copy this checklist and tick it off:
```
Retrospective progress:
- [ ] 0. Confirm the server answers
- [ ] 1. Read the team's rules
- [ ] 2. Read git for what the period went into
- [ ] 3. Separate what the code did to the score from measuring
- [ ] 4. Read where the code stands
- [ ] 5. Write from the template
- [ ] 6. Verify, then write the lines for the person who asked
```

### 0. Confirm the server answers
If the cca tools are deferred, load them once by exact name:
`select:mcp__cca__get_trend,mcp__cca__audit,mcp__cca__list_findings,mcp__cca__report_platform_issue`. If they are missing or
ask you to sign in, follow the signing-in rule. Call `cca:get_trend` with no arguments. If it has no trend point, write from
git, and under "Where the code stands" write "There is no scan of this repository yet."

### 1. Read the team's rules
Read CLAUDE.md or AGENTS.md, CONTRIBUTING, the README and docs/ (SOPs, runbooks, ADRs), and note what they ask of item titles,
gates, tests first, commits and ADRs.

### 2. Read git for what the period went into
Read `git log --since=<start> --until=<end>` (or the tag range) with the commit bodies, and `git diff --shortstat` for the
period. From them:
- Count commits per day, name the days with none, and count the conventional-commit types. Group the work into themes by item
  or area in time order, and say how much of it is product work.
- List as bugs the `fix` commits whose body describes a defect in behaviour, not a test, doc or style repair.
- List what did not go well: reverts, repairs of what an earlier commit broke, one day far busier than the rest, one commit over
  hundreds of files, a catch-all item, commits that only recount or regenerate what a script could, work against a team rule.
- List what went well: habits the history shows, each with two or three commit ids.
- Before you say a gate was not run, grep the bodies for its record (`--complete`, "gated", "green").

### 3. Separate what the code did to the score from measuring
Keep the trend points and marks inside the period. What the code did is the last score minus the first, minus each mark's
`headlineDelta`, minus each unexplained move (for the whole trend: `codeMove.byTheCode`). Credit each lens value move of 3 or
more by the crediting rule; one no commit explains is an open question. Report each unexplained move with
`cca:report_platform_issue` (`area` "scan-pipeline", `title` "Unexplained trend move", `description` with both dates,
commits and scores); if it cannot be called, its text goes to the person in step 6, never into the retrospective.

### 4. Read where the code stands
Call `cca:audit` with no arguments; read only `scan`, `health`, `band`, `queue`, the first three `plan` entries, the audit lens
entries and `gapReports`.
- Say a security dimension absent from the audit lens entries, or "not measured" in `gapReports`, is not measured, never clean.
- For a security dimension below 10, call `cca:list_findings` (`dimension`, `state` "open") for the count and files; write none
  if all await a ruling.
- When `queue.resolutionsRefused` is above 0, put the fixes that did not hold under "What did not go well" with their files.
- When `queue.decidedCauses` is above 0, call `cca:list_findings` (`disposition` "Accepted", `limit` 5): its files are evidence
  for a code try (one repair repeated across screens), never a try about the scan. Make a `plan` entry a try only when it is
  real work now.

### 5. Write from the template
Write from the template. In the sections after "Things to try", refer to a try ("see try 2") instead of repeating it.

```markdown
# Retrospective, <start> to <end>

Sources: `<git range>` up to `<commit>` (<date>), and the scan of <date> (`<commit>`).

## At a glance
| | |
|---|---|
| Commits | <n> non-merge, <m> merges |
| Days with commits | <d> of <days>: <date (n)>, ... **<busiest (n)>** |
| Kind of work | <n> refactor, <n> test, <n> fix, <n> feat ... |
| Size | <files> files, +<a> / -<b> lines (net over the period) |
| Items | <n> distinct; <n> cite a number with no title |
| Score | <first> (<date>) to <last> (<date>): the code did <x>, <k> changes in measuring <y>[, unexplained <z>] |

<Two sentences: what the period went into, how much was product work, and the one thing that stands out.>
## What we did
- **<Theme> (<dates>).** <What changed for the code>, <items>, <two or three commit ids>.
## Bugs found along the way
- <The defect, in the user's terms> (`<commit>`, <item>).
## What went well
- **<Habit to keep>.** <Evidence, with commit ids.>
## What did not go well
- **<What cost>.** <Evidence and what it cost>, or "see try <n>".
## Where the code stands (scan of <date>)
- **Score:** <score>, band <band>; <fixes that did not hold, or "every reported fix held">.
- **Security:** <open findings or "none open">. **Not measured:** <each dimension by its area>: unknown, not clean.
- **What the code did / Measuring:** <see example 1>.
## Things to try next sprint
1. **<The try.>** <Evidence, counts, commit ids.> Try: <action>. Check: <what the next retro will see>. (three to seven)
## Open questions for the team
- <What history cannot settle: whether the pace was planned, a lens move no commit explains.>
```

### 6. Verify, then write the lines for the person who asked
Verify each line against this list, correct each failure, and verify again:
- every count recounted; merges apart; per-day counts sum to the total; lines from the net diff;
- every lens value move credited only to commits in its window that touch its area; none to a mark or an unexplained move;
- every security dimension not measured said so; every bug a defect in behaviour; no section repeats a try; every team rule
  cited exists;
- every try has evidence and a check, and is work for the team; no "I", no person blamed.

Then, below the retrospective and to the person who asked (not the team), write: the default you took; if
`cca:report_platform_issue` could not be called, "Please send this to the service: Unexplained trend move, <date A>
`<commit>` <score> to <date B> `<commit>` <score>, no commit and no mark between them."; last, a short closing paragraph as
the server's instructions describe, from the `contribution` items that changed this retrospective, leading with what it would
otherwise have got wrong.

## Examples

1. Under "Where the code stands", for the team:

> **What the code did:** the score went from 71.2 to 84.0; the code did +4.1, three changes in measuring did +8.7, and the fall
> to 62 on 9 May is one of them, not a regression. Security rose 6 at the scan of 7 May (`a41c0d2`); the only dependency commit
> in that window is `9e1b3f0`, so likely from that upgrade. The contrast fixes (`c2d7e81`, 8 May) came after that scan.

2. Under "Things to try next sprint":

> 2. **Stop filing work under one catch-all item.** 41 non-merge commits went under #212 "cleanup", 12 with no title, though
>    CLAUDE.md asks for one; from 6 May the titled items #230 to #241 carried 1 to 3 commits each. Try: one item per
>    cluster, filed before the work starts. Check: no item carries more than ten commits.
