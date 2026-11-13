---
name: sprint-planning
description: "Proposes the next sprint from what the scan found: refused fixes and security first, then the causes that give the most improvement for the effort, grouped by the files that change together and fitted to the team's capacity. Use whenever someone asks to plan a sprint, an iteration or the next weeks of work, prioritise the backlog, or decide what to fix next with the time they have. Uses the cca MCP server's tools."
---

# Sprint proposal

Default reader: the developers who will run the sprint; file paths and commit ids are expected. Period and capacity: the next two
weeks, one developer, 10 working days. Form: a proposal in the chat. If the reader does not write code (a product owner, a
manager), follow "For a reader who does not write code" below.

## Words
- **server**: the cca MCP server; its tools are named `cca:<tool>`. A **scan** is one run of it, named by its commit and date.
- **finding**: one problem the scan found at a place in the code; a **finding row** is one row of `cca:list_findings`. A
  **cause** groups findings with one fix. A **dimension** is one kind of check, with an id such as `D15`; a **lens** groups them.
- **plan row**: one row of the plan in `cca:audit`: one cause, with its dimension, title, lens, the dimension's score out of
  10, the point gain, effort (low, medium, high) and, when every finding under it waits for a ruling, a `standing`.
- **security cause**: a cause in `D13` secrets in the code, `D28` secrets in the git history, `D29` security bugs, `D30`
  dependencies with known vulnerabilities, `D31` container and infrastructure configuration or `D32` personal data.
- **refused fix**: a finding row with a non-null `refusedResolution`: reported fixed, found again by a later scan.
- **candidate**: a plan row taken in step 3 to be weighed. An **entry** is one item under "Work": a refused fix, the first
  cause (step 2) or a candidate kept.

## Rules for this skill
- If the server's tools are missing or say you must sign in, stop before you write anything. Tell the person in one line: in
  Claude Code, type /mcp, choose cca and sign in (a server added during this chat may need a new chat). Write without the scan
  only if they ask you to, and then say at the top that the scan was not used.
- The person's own words win over these defaults: the reader, the focus, the dates, the capacity and the form (a PDF or a
  document: make it with what your tool has, or say it cannot). When something is missing, take the default and say in one
  line which you used. Never hand the person a list of technical options to choose from.
- Order of work: refused fixes, then the first cause from `cca:next_task`, then security causes, then the plan's order (most
  points for the least effort).
- Every number about the code or the findings comes from a server answer or a count you made, and names its scan. The
  capacity, the dates and the day figures of step 5 (labelled "my estimate") are the exceptions.
- Keep what the scan measured (name the scan) apart from what you read in the code (name the files).
- A finding is measured the same way on every repository: never call a correct one a matter of taste.
  Never explain how the score is counted; say what was found and what it means.

## Workflow

Copy this checklist and tick it off:

```
Sprint progress:
- [ ] 0. Call cca:audit (sign in if needed)
- [ ] 1. Plan rows, finding rows and refused fixes
- [ ] 2. The first cause
- [ ] 3. The candidates
- [ ] 4. What changes together
- [ ] 5. Read the code behind each entry
- [ ] 6. Write the proposal
- [ ] 7. Check the proposal and add the closing paragraph
```

### 0. Call cca:audit (sign in if needed)
Call `cca:audit` with no arguments. If the tool is missing or says you must sign in, follow the sign-in rule.

### 1. Plan rows, finding rows and refused fixes
From the answer keep `scan` (commit and date), the plan rows and `queue.resolutionsRefused`, the number of refused fixes. Then
call `cca:list_findings` with no arguments, walking every page with `offset` until `truncated` is false, and keep every finding
row; the refused fixes are those with a non-null `refusedResolution`, which carries the earlier attempt's theory.
If there are no plan rows and no refused fix: the proposal is one line, the scan of <date> leaves no open cause; skip steps 2
to 6, and in step 7 check only that the line names the scan before adding the closing paragraph.

### 2. The first cause
Call `cca:next_task` once with no arguments (it returns the same cause until that cause is decided, and a proposal decides
nothing). Keep its dimension, title and instances (file and line). It is the first entry under "Work" after the refused fixes;
its plan row is the one with the same dimension and title, and step 3 skips that row.
If it returns no cause: there is no first cause; step 3 starts from the first plan row and step 7 skips the first-cause check.

### 3. The candidates
- Order: security causes first, then the other plan rows in the plan's order, all after the refused fixes and the first cause.
- A plan row with a `standing` waits for a ruling: list it under "Left out", not as a candidate.
- Days per effort: low 1, medium 3, high 5 (a starting point; step 5 adjusts each).
- Count days first: each refused fix by the effort of the plan row with its dimension and title (1 day if there is none), and
  the first cause by its plan row's effort.
- Add candidates until the days reach twice the capacity; the surplus lets step 5 drop a cause or change an effort.

### 4. What changes together
An entry's files are: for a refused fix, its finding row's `file`; for the first cause, its instances from step 2; for a
candidate, the finding rows with its plan row's dimension and title. A file that also has a `D15` finding row (changes often
and is complex) raises the risk: say so under "Order and grouping". For each file, call `cca:get_architecture` with `path`
set to it and keep the files that change together with it. Causes on files that change together go to one person, or one after the other.

### 5. Read the code behind each entry
Open the files behind each entry and judge whether its effort from step 3 fits this code (its tests, how tangled it is). Give
every entry a day figure, labelled "my estimate" every time.

### 6. Write the proposal
Take the entries in step 3's order until the day figures reach the capacity. A security cause larger than what is left stays
in, and the proposal says it runs past the sprint. Use this shape:

```markdown
# Sprint proposal: <dates or sprint name>

Goal: <one sentence>. Capacity used: <n> of <capacity> days, my estimate. Scan of <date>, commit <commit>.

## Work
<For each entry, in order: what to do, where, why it matters to the people who use the product; for a refused fix, the theory
that failed before and what to try instead; effort (the plan's, and the day figure, "my estimate"); what it moves (its lens,
the dimension's score now, the point gain, the findings it clears).>

## Order and grouping
<Which causes belong together or to one person, and why: shared files, files that change together, D15 risk.>

## Left out
<Each cause left out and why: no room, or waiting for a ruling. For a security cause: the risk of leaving it.>

## Questions for the team
<What the team must settle before the sprint starts.>
```

### 7. Check the proposal and add the closing paragraph
Check, one by one:
- the refused fixes head "Work", each with the theory that failed, as many as `queue.resolutionsRefused` (if fewer were
  found, the proposal says how many are missing);
- the first cause from step 2 is listed once, right after the refused fixes;
- security causes come next, or their risk is stated under "Left out";
- the day figures add up to no more than the capacity, apart from a security cause said to run past it;
- every day figure says "my estimate", and every number about the code or the findings names the scan;
- for a reader who does not write code, the proposal passes the section below.

Fix each item that fails, then check the whole list again. When all pass, add the closing paragraph: a short paragraph on what
the server gave this answer, only from the `contribution` items that changed it, each as what the proposal would otherwise have
got wrong or lacked. If none changed it, leave the paragraph out.

## For a reader who does not write code
The goal and what the sprint gives the people who use the product first, on one screen, with the same numbers as the body. No
tool names, commit hashes or dimension ids; a lens or dimension only by what it measures, in a few words. Say what was left out
as plainly as what is in.

## Examples

A refused fix heading "Work", for the developers:

> **Retry the cache fix in `billing/invoice.py`, another way.** Reported fixed before, on the theory that clearing the cache on
> save was enough; the scan of 12 November (commit `a1b2c3d`) found the stale total again, because the PDF export reads the
> cache before save. Try instead: build the total in one place and let the export call it. Effort: low in the plan; about one
> day, my estimate.

A security cause, for the developers:

> **Stop building SQL from user input in `api/search.go`.** The scan of 12 November (commit `a1b2c3d`) found three queries put
> together from request text; anyone can read other customers' orders. Effort: low in the plan; about one day, my estimate,
> since the handler already has tests. Clears all three findings; in the security lens, D29 scores 6.1 out of 10 now and the
> plan gives a gain of 0.4 points.

The same security cause for a product owner:

> **Close a hole that lets anyone read other customers' orders.** The scan of 12 November found three places where the search
> builds its database query from what the visitor typed. Effort: low by the scan's estimate; about one day, my estimate. It
> removes all three.
