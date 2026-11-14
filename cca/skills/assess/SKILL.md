---
name: assess
description: "Writes an assessment of a codebase from a reading of the repository and its scan: how good it is, whether it can go live, its strengths and weaknesses, the risks and what they cost, how it is moving and what to fix first, for the person, their manager, a board or a buyer. Use when someone asks how good, healthy, solid or risky the code is, wants a review of the whole project, a quality report, a technical assessment or due diligence, or asks 'what state is this in', even without mentioning a scan. Uses the cca MCP server's tools for the facts reading cannot give."
---
# Codebase assessment

In one read the reader can decide the code's state, if it can do its job (go live, be sold, be handed over), which risks carry a
price, how fast it improves or worsens and what to do first. Reading carries the verdict; the server adds what reading cannot.

## Words
- **server**: the cca MCP server; its tools are `cca:<tool>`. A **scan** is its analysis of one commit, named by commit and date.
  A **finding** is one problem at one place; a **cause** groups findings with one fix (with a `standing` it is already decided and
  carries no work). The **plan** is `plan` in `cca:audit`: causes ranked by points gained per effort.
- **dimension**: one kind of measurement (id such as `D3`; in plain words its `whatItMeasures`) with a **dimension score** out of
  10; a **lens** groups dimensions. The **score** alone is the overall figure out of 100 (`health`), in **points**.
- **security dimensions**: `D13` secrets in the code, `D28` secrets in the git history, `D29` security bugs, `D30` vulnerable
  dependencies, `D31` container and infrastructure settings, `D32` personal data.
- **repaired files**: the files `D15` lists as fixed again and again. A **trend entry** is one scan on `cca:get_trend`'s trend; a
  **mark** is one where the measuring changed. An **item** is one statement in the document: a strength, problem or action.

## Rules
- **Signing in.** If the server's tools are missing or say you must sign in, stop before you write anything and tell the person in
  one line: in Claude Code, type /mcp, choose cca and sign in (a server added during this chat may need a new chat). Write without
  the scan only if they ask, and then say at the top that it was not used.
- **Defaults.** The reader is the person who asked, the form Markdown in chat, the period the latest scan (plus the trend and all
  of git), the length under 70 lines for a manager or board, 100 for a developer, 140 for a buyer. The person's own words win over
  these defaults: the reader, the focus, the form (a PDF or a document: make it with your tools or say you cannot). For anything
  unsaid, take the default, write, and name it in one line. Never hand them a list of technical options to choose from.
- **Every claim is verified.** Every count comes from a command you ran (write its exact result) or a named scan; a figure that
  can be counted two ways (test lines, merges, people) says once what was counted and is the same everywhere. Every file named
  exists at that path; a claim about what code does comes from reading it; dates come from git, never memory. When sources
  disagree (a README against the code, an ADR against a newer doc), the code or config decides: say which is right.
- **Scan and reading.** End each item with "(scan)", "(reading: <what was read>)" or "(both)": files or a command for a developer,
  plain words for anyone else ("the start-up code"). A finding or dimension score your reading contradicts (access rules rated
  high where no sign-in exists): say so and call `cca:dispute_finding` with the evidence. Write a problem the scan missed, or
  files it did not count, as a fact and report it with `cca:report_detector_gap`. Never call a correct finding a matter of taste.
  Never explain how the score is counted; say what was found and what it means. A strength needs evidence from reading too.
- **Order.** Anything that stops the code doing its job (it cannot go live, a security finding, an ownership gap for a buyer)
  comes first, from the scan or reading; then the plan's causes that carry work, in its order; then reading-only items.
- **Voice.** Write to the reader about the code: no "I" or "my count", no scan-output framing; name a scan only beside its number.
- **Reader.** Never present a mark's move as the code's. A developer: files, dimension ids, commands and next steps; each mark in
  its own sentence. Anyone else: the trend in one sentence, "the code itself moved the score by X; changes in how it is measured
  account for the rest". **Management or a board:** decisions, risk, cost of waiting, what to ask the team, one figure to quote
  upward; no tables of commit kinds, tool names, commit hashes, dimension ids or namespace counts; a dimension only by what it
  measures; nothing on the scanner. **A buyer:** what they inherit, what transfers, the risks that carry a price, what the seller
  must fix, disclose or hand over, what their engineers should run before signing; disputes in one sentence on what they confirm.

## Workflow
Copy this checklist and tick it off:
```
Assessment progress:
- [ ] 0. Load the tools and call the audit
- [ ] 1. Read the repository's own sources
- [ ] 2. Count size, activity and people with git
- [ ] 3. Read whether the code can do its job
- [ ] 4. Fetch only the server facts the document will print
- [ ] 5. Read the code behind the findings
- [ ] 6. Write from the template
- [ ] 7. Review the document, then write the lines for the person who asked
```

### 0. Load the tools and call the audit
Load in one search by exact name (`mcp__cca__<tool>` in Claude Code): `cca:audit`, `cca:get_dimension`, `cca:list_findings`,
`cca:get_trend`, `cca:get_architecture`, `cca:dispute_finding`, `cca:report_detector_gap`, `cca:report_platform_issue`. Confirm
the result names each one; note any missing for step 7. Call `cca:audit` with no arguments; if it fails, follow the Signing in
rule. Keep `scan` (commit, `scannedAt`), `health` and `band`, each lens score, each dimension score and `whatItMeasures`, `plan`,
`inventory`, `gapReports` and `contribution`.

### 1. Read the repository's own sources
Read its docs, ADRs, runbooks, build, CI and licence files, and extract what the product is and for whom, its stack and parts,
what is built and what is only designed, the gaps the team has written down, how it is tested and deployed, and what it depends on
outside the repository (CI runners, hosting, secrets, gitignored local state, predecessor projects).

### 2. Count size, activity and people with git
- **Size.** Count lines and files by kind with `git ls-files` (production source, markup and templates, tests, data; tests where
  test projects really live, also inside source folders). Name any kind the scan's `inventory` did not count. Find the largest
  files of one kind with `git ls-files -z -- '*.<ext>' | xargs -0 wc -l | grep -v ' total$' | sort -rn | head -5`.
- **Activity.** Count commits by month with `git log --format=%ad --date=format:%Y-%m | sort | uniq -c` and name empty months.
  Compare the first commit's date and size (`git show --stat <root>`) with the docs' dates; docs that start earlier mean history
  was not handed over. List the tags. Say the last 90 days' commits by kind in one sentence ("mostly fixes and restructuring").
- **People.** Run `git shortlog -sn --all`; one or two people with most commits is a key-person risk. Give counts, not emails.
- **Busiest folder.** If git fails (no history), say activity is not measured; otherwise run
  `git log --since="90 days ago" --name-only --format= | awk -F/ 'NF>1{print (NF>2 ? $1"/"$2 : $1)}' | sort | uniq -c | sort -rn | head -5`.
- **Tests.** Never build, test or deploy. Give the result of the last recorded full test run (CI or committed results) and the
  commits since it, and say plainly that nothing was run for this assessment.

### 3. Read whether the code can do its job
Read the entry points and the start-up and configuration code: is sign-in implemented (search the identity and auth wiring and
what implements its interfaces, not only the interfaces), does start-up refuse production for something missing, are features
stubbed or "not implemented"? For a buyer, also read for licence and copyright, content or rules from another product, predecessor
projects, AI co-author trailers and the ownership question they raise, dependency licences, what does not transfer.

### 4. Fetch only the server facts the document will print
- **Weak spots.** Call `cca:get_dimension` for the three lowest dimension scores that carry findings or a `scoreNote`. Print each
  `peers` median beside its dimension score, or once "no peer comparison yet" when withheld; with `reliability.measured` false,
  present its findings as "leads to verify". A low dimension with nothing to act on gets one line for a developer, none otherwise.
- **Strengths.** Use `cca:get_dimension` on up to two 10s in the top two lenses; quote `measured` ("2.1 % duplicated"), not 10.
- **Security.** Read the security dimension scores and `gapReports`; for one under 10, call `cca:list_findings` with `dimension`
  and `limit` 5. A security dimension that `gapReports` names as not measured did not run: say so and whether CI covers it.
- **Repaired files (every reader).** Call `cca:list_findings` with `dimension` "D15" and `limit` 5, first `state` "open", then
  `state` "blocked" for findings a maintainer confirmed after a dispute (`maintainerNote`), the strongest evidence of a hotspot.
- **Structure (buyer, developer).** Buyer: call `cca:get_architecture` with no arguments for `moduleCount` (write "N namespaces",
  never projects or folders), `cycles` and context edges that breach a boundary. Developer: call it with `path` the top repaired
  file, only if its change partners will be named.
- **Trend.** Call `cca:get_trend` for `codeMove` and each mark's `at`, `sentence` and `headlineDelta`; under two trend entries,
  say the movement is not yet measured. A score move between two neighbouring trend entries with no commit and no mark between
  them (an empty `git log <a>..<b>`; if git cannot list them, say the move is not verified) is unexplained: take it out of the
  code's move, and report it with `cca:report_platform_issue` (`area` "scan-pipeline", `title` "Unexplained trend move", both
  dates, commits and scores). Before writing which part was larger, compare the numbers.

### 5. Read the code behind the findings
Open the files behind the security findings, the top of the plan, the weak dimensions and the repaired files: does each finding
hold in context, and what does a fix take? In the largest and the repaired files, look for a defect no scan sees (state not reset
on an early return, a shared lifetime, a comment that contradicts the code beside it) and give it with file and line.

### 6. Write from the template
Leave out an empty section; a buyer adds "What you would inherit" and "Before signing".
```markdown
# <Project>: assessment

<Verdict in three sentences: the state, whether it can do its job today and what stands in the way, the score with its scan.>

| Area | State |
<Five to seven rows: quality, security, can it go live, people, movement; a buyer adds ownership and transfer.>

## What holds
<Strengths with evidence from reading and what the scan measured.>

## What holds it back
<In the Order rule's order; each with where it is, what it costs and what to do; a weak dimension's peers median.>

## How it is moving
<Activity by month and what recent work was; the code's move apart from marks and unexplained moves; the repaired files.>

## What to do first
<Three to five actions, in the Order rule's order.>

## What was not measured
<Short: security dimensions that did not run, leads with unmeasured reliability, what was not built or run.>
```

### 7. Review the document, then write the lines for the person who asked
Review the document against the Every claim is verified, Scan and reading, Order, Voice and Reader rules and the length, and
confirm that the verdict, table and trend sentence agree with the body's numbers. Correct and review again until nothing fails.

Then, after a `---` line, write for the person who asked, not the reader, saying so in its first words
("For you, not for <reader>:"), in at most six lines: the default you used, if any; each report filed, or, for a tool missing or
refused, its ready title and text to send; and a short paragraph on what the server gave, only from `contribution` items that
changed this assessment, each as what it would otherwise have got wrong or lacked, with its number or file. If none did, omit it.

## Examples
For management, the verdict and one item:
> The code is in good shape (82 of 100, scan of 12 November), but it cannot go live yet: there is no sign-in, and the application
> is built to refuse to start in production until there is (reading: the start-up code). Building sign-in is the main work before
> launch. Over the last month the team's own changes raised the score by 4 points; quote that, not the jump on the chart.
>
> **What holds it back, 2.** One person made 1,904 of the 1,911 changes (reading: the commit history). Ask for a second reviewer
> on the busiest areas before the launch date is set.

For a developer, one item:
> **What holds it back, 3.** `D3` 3/10 (scan `4f1c9a2`, 12 Nov; peers median 6/10): `src/billing/InvoiceEngine.cs` is 2,140 lines
> with 61 methods and was repaired 7 times in 90 days (both). Split pricing from persistence first; the plan puts the gain at
> 4.2 points for medium effort.

