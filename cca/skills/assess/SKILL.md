---
name: assess
description: "Writes an assessment of a codebase from its scan and a reading of its code: how good it is, its strengths and weaknesses, the risks, how it is moving and what to fix first, for the person, their manager, a board or a buyer. Use whenever someone asks how good, healthy, solid or risky the code is, wants a review of the whole project, a quality report, a technical assessment or due diligence, or asks 'what state is this in', even without mentioning a scan. Uses the cca MCP server's tools."
---

# Codebase assessment

**Defaults.** Reader: the person who asked; when they do not write code, follow "For a reader who does not write code" below.
Period: the latest scan, with the trend for the movement. Form: Markdown in the chat. When something is missing, take the
default, do the work and say in one line which default you used; never hand the person a list of technical options to choose
from. The person's own words win over these defaults: the focus, the form (a PDF or a document: make the file with what your
tool has, or say it cannot) and the reader, written for like this:

- **Management:** decisions, risk, the cost of waiting, what to ask the team.
- **A buyer doing due diligence:** what they would inherit, the risks that have a price, what the seller should fix or disclose.
- **A developer on the team:** files, dimension ids and next steps.

**Words.** The **server** is the cca MCP server (tools `cca:<tool>`). A **scan** is one run of it, named by commit and date. A
**finding** is one problem at one place; a **cause** groups findings with one fix. A **dimension** is one kind of check (id such
as `D15`); a **lens** groups dimensions; the **headline** is the overall score. The **plan** is the ranked list of causes in
`cca:audit`, each with its point gain and effort. A cause **carries work** when it has open or blocked findings; one with
a `standing` carries none (every finding under it waits for a maintainer's ruling). **Security findings** are those of `D13` (secrets in the code), `D28` (secrets
in the git history), `D29` (security bugs), `D30` (vulnerable dependencies), `D31` (container and infrastructure configuration)
and `D32` (personal data). A **mark** is a point on the trend where the way of measuring changed.

## Rules for this skill

- **Scan and reading apart.** The scan has read every file, the history and the vulnerability databases: use it to choose where
  to look, then read. Label each point "scan" (name the scan), "reading" (name the files) or "both".
- **Numbers.** Every number comes from the server's answer or a count you made with git or the files, and names its scan or
  date. A claim about what a test or function does comes from reading what it calls; if you did not read it, say so.
- **Findings.** A finding your reading shows is wrong: say so and call `cca:dispute_finding` with your evidence. A problem the
  scan missed: report it as yours with `cca:report_detector_gap`. Never call a correct
  finding a matter of taste. Never explain how the score is counted; say what was found and what it means.
- **Summary.** `summary` from `cca:get_summary` is true wherever it is printed: build on it, never contradict it. Use its
  `vocabulary` to explain a dimension or kind of finding, instead of your own words.
- **Peers.** `peers` from `cca:get_dimension` is never part of the score; quote its median beside a percentile, never a
  percentile alone. When `reliability.measured` is false, present that dimension's findings as neither certain nor noise, e.g. "(scan; how
  often this check is right is not yet measured, so these are leads to verify)".
- **Trend.** Each mark's own move is its `headlineDelta`; what the code did is `codeMove`. Never present a mark's move as the
  code's; give each mark in its own sentence. A headline move between two neighbouring trend points with no commit and no mark
  between them (an empty `git log <commit a>..<commit b>`; if git cannot list them, say the move is not checked) is unexplained: keep it out of what the code did and report it with
  `cca:report_platform_issue` (`area` "scan-pipeline", `title` "Unexplained trend move", both dates, commits and headlines).
- **Order.** The plan's order: most points for the least work.

## Workflow

Copy this checklist and tick it off:

```
Assessment progress:
- [ ] 0. Check the server
- [ ] 1. Audit and summary
- [ ] 2. Weakest dimensions and strongest lenses
- [ ] 3. Trend
- [ ] 4. Architecture
- [ ] 5. Security findings and the top of the plan
- [ ] 6. Read the code
- [ ] 7. Write the assessment
- [ ] 8. Check the assessment
```

**0. Check the server.** Step 1's `cca:audit` is the probe. If the server's tools are missing or say you must sign in,
stop before you write anything and tell the person in one line: in Claude Code, type /mcp, choose cca and sign in (a server added
during this chat may need a new chat). Write without the scan only if they ask, and then say at the top that it was not used.

**1. Audit and summary.** `cca:audit`: keep `scan` (commit and date), the headline, each lens score and the plan. Then
`cca:get_summary`: its `summary` is the base of the verdict.

**2. Weakest dimensions and strongest lenses.** Weakest: the three scored dimensions with the lowest /10 in the audit; call
`cca:get_dimension` with each id and keep `peers` and `reliability.measured`. A weak dimension with no findings measures
something missing: `cca:get_task` with its id; its `action` says what. Strongest: the two lenses with the highest score, and in
them up to three dimensions scoring 10 (if none scores 10, their highest-scoring dimensions), each with `cca:get_dimension` so a strength has evidence too.

**3. Trend.** `cca:get_trend`: keep `codeMove` and each mark's `at`, `sentence` and `headlineDelta` for "How it is moving". With
fewer than two trend points, say the movement is not yet measured.

**4. Architecture.** `cca:get_architecture` with no argument: keep the modules, their layers and every cycle. If it returns no
modules, write "not measured" under Structure.

**5. Security findings and the top of the plan.** `cca:list_findings` with `dimension` set to each of `D13`,
`D28`, `D29`, `D30`, `D31` and `D32` in turn, adding up `totalMatching`, not the rows returned. Then the plan's first five causes
that carry work.

**6. Read the code.** Open the files behind the security findings, the top of the plan and the weakest dimensions' findings: does
each hold in context, and what does a fix take? Then read the entry points and the busiest folder, the first line of:

```bash
git log --since="90 days ago" --name-only --format= | awk -F/ 'NF>1{print (NF>2 ? $1"/"$2 : $1)}' | sort | uniq -c | sort -rn | head -5
```

If git fails (no history), say the busiest folder is not measured and read the entry points only. Judge what a scan cannot measure: design, intent, naming, whether the tests check behaviour, documentation.

**7. Write the assessment** in this shape, labelling each point "scan", "reading" or "both":

```markdown
# <Project>: assessment

<Verdict in three sentences, with the headline and its scan.>

## What holds
<Each strong lens and dimension with its evidence; a module layering that holds.>

## What holds it back
<Security first, then the plan's order; each with where it is and what to do; a weak dimension's peers median.>

## Structure
<The modules and layers in one line; each cycle with the modules in it.>

## How it is moving
<The code's move from codeMove; each mark in its own sentence.>

## What to do first
<Three to five actions in the plan's order.>

## What was not measured
<What neither the scan nor your reading covered; any dimension with reliability.measured false.>
```

**For a reader who does not write code:** the verdict first, on one screen, with the same numbers as the body; no tool names,
fingerprints, commit hashes or dimension ids, and a lens or dimension only by what it measures, in a few words; say what was not
measured as plainly as what was.

**8. Check the assessment.** Check: the verdict uses the same numbers as the body; every number names its scan or your count;
every file named exists; each point is labelled scan, reading or both; no mark's move is told as the code's; each value kept in
steps 1 to 5 appears in the assessment, in the form the reader rules allow (for a non-coder: by date and in plain words, no
commit or dimension id); the detail fits the reader. Fix what fails and check again.

**Closing paragraph.** End with the closing paragraph the server's instructions describe, from the `contribution` items that changed this answer.

## Examples

For management (does not write code), the verdict, one point and the closing paragraph:

> The code is in fair shape: 71 of 100 on the scan of 12 November. Two security problems need fixing before the next release;
> the rest is steady work for the coming months. Over the last three months the team's own changes raised it by 4 points.
>
> **What holds it back, 1.** Customer passwords are checked in a way an attacker can time (both: the scan, and reading the login
> code). It is a one-day fix; ask the team to schedule it before the next release.
>
> Without the scan, this assessment would have missed the password check, which sits in a file nobody has changed this year.

For a developer, one point:

> **What holds it back, 2.** `D3` 3/10 (scan `4f1c9a2`, 12 Nov; peers median 6/10): `src/billing/InvoiceEngine.cs` is 2,140 lines
> with 61 methods (both). Split pricing from persistence first; the plan puts the gain at 4.2 points for medium effort.
