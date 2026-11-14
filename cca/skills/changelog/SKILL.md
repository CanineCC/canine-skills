---
name: changelog
description: "Writes a changelog from the git history for the people who maintain the code: Keep a Changelog sections with a commit on every entry, what maintainers must now use or respect, security with advisory ids, and a short line on how quality moved. Use when someone asks for a changelog, a CHANGELOG entry, a list of changes, what changed or what was shipped since a date, tag, release or sprint. For customers or users, use release-notes. Uses the cca MCP server's tools."
---

# Changelog

**Defaults.** Reader: the maintainers, so commits, files and dimension ids stay in; afterwards they find every change and its commit, know
what now binds them, see security fixes with advisory ids and what is still open, and know the code's share of the overall score's move. A
changelog lists changes, not a scan report. If the reader is customers or users, use the release-notes skill instead. Period: since the last tag, or, with no tag, over the last two weeks ("the last sprint" too, unless a tag or date
says otherwise). Form: Markdown in the chat; for a PDF or a document, make the file with the tool you have or say you cannot. A **named
path** limits it to that path. The person's own words win over these defaults. If something is missing, take the default, do the work and
say in one sentence which default you took; never hand the person a list of technical options to choose from.

**Words.** The **server** is the cca MCP server; its answers can carry `contribution` items, each saying what it added. A **scan** is one
analysis of the repository by the server, named by the commit it read and its date. A **trend scan** is one scan listed by `cca:get_trend`
(the **trend**); its `headline` is the **overall score** (0 to 100). A **dimension score** (out of 10) is a dimension's `score` in
`cca:audit`'s lenses. A **mark** is one item of the trend's `marks`: a change in how the overall score is measured, with its own move
`headlineDelta`. An **unexplained move** is a `headline` difference between neighbouring trend scans with no commit and no mark between them
(`git log <commit a>..<commit b>` is empty, no mark's `at` lies between their dates). The **code move** is what the code alone did to the
overall score. The **security dimensions** are those in `cca:audit`'s Security lens plus the secret dimensions `D13` (code) and `D28` (git
history); name each in plain words by its `name` in `cca:audit` ("vulnerable dependencies"). **Shared code** is a function, type or module
introduced for other code to call instead of copying. An **entry** is one changelog bullet.

## Requirements

1. **Git history and scan apart.** What changed and what was fixed come from the git history; what is still open, from the scan.
2. **Every entry for a change cites the commit that made it** (short hash, one per named thing). Issue numbers go beside the hash, never
   instead of it: the number of the item the commit closed, not of the one where the defect began.
3. **Numbers** come from a git command you ran or a server answer, never from counting by hand; count merges apart.
4. **Moves.** Never give a mark's move or an unexplained move as the code move.
5. **Only what the range changed.** A change reverted inside the range is one Reverted entry, never also a Changed one; a test that pins
   existing behaviour is not a change in behaviour; "kept on purpose" only where both copies still exist at HEAD.
6. **Neutral voice.** No "I", no "my count"; no tool names, server notes or failed calls inside the changelog.
7. **Findings** are measured alike on every repository: never call a correct one a matter of taste. Never explain how the overall score is
   counted; say what was found and what it means.

## Workflow

Copy this checklist and tick it off:

```
Changelog progress:
- [ ] 0. Load the tools and read the trend
- [ ] 1. Set the range and read the repository's conventions
- [ ] 2. Read the git history
- [ ] 3. Sort the changes
- [ ] 4. Work out the code move over the range
- [ ] 5. Read what the scan says now
- [ ] 6. Write the changelog
- [ ] 7. Check the changelog
```

**0. Load the tools and read the trend.** Load `cca:get_trend`, `cca:audit`, `cca:list_findings`, `cca:get_dimension`,
`cca:get_architecture` and `cca:report_platform_issue` in one tool search, note which loaded, and call `cca:get_trend`. If the tools are
missing or say you must sign in, stop before you write anything and tell the person in one sentence: in Claude Code,
type /mcp, choose cca and sign in (a server added during this chat may need a new chat). Write without the scan only if they ask, saying so
at the top. If the call fails otherwise, skip step 4 and write in the Quality paragraph that the trend was not available.

**1. Set the range and read the repository's conventions.** Start at the last tag (`git describe --tags --abbrev=0`), else at the first
commit of `git rev-list --since="2 weeks ago" HEAD`; note the first and last commit and dates. When the period starts before its first
commit, the header gives the period and the summary the first commit's date. Follow the format of a `CHANGELOG*`, ready to paste at its top;
with none, say so in one sentence. Read what `CLAUDE.md`, `AGENTS.md` and `CONTRIBUTING*` say on commits, issue numbers and decisions; if
issue numbers are local or not shared, say so once in the summary.

**2. Read the git history.** Read `git log --no-merges --format='%h %ad %s%n%b' --date=short <from>..HEAD` (add `-- <path>` for a named
path); the bodies hold causes, revert reasons and the rules that bit. Count commits with `git rev-list --count --no-merges <from>..HEAD` and
`--merges`, files with `git diff --shortstat <from>..HEAD`. With no commits, say so in one sentence and stop. Read the diffs of every
dependency, security and configuration change (old and new versions, advisory ids), every fix, revert and removal, and the five largest
commits by lines changed in `git log --no-merges --shortstat` (they carry most of the work); read each changed ADR's decision.

**3. Sort the changes** into Added, Changed, Fixed, Removed and Security (by kind of change only), grouping related commits into one entry.
A Fixed entry says what was wrong, why, and what it does now, in one or two sentences; fixes a person using the software would notice come
first. Then gather **For maintainers**: shared code and what it replaces, rules now enforced with the test that fails on a breach, decisions
recorded or amended, copies kept on purpose, reverts with their reason, new commands or runbooks, each citing the commit that introduced it
(`git log --reverse --format=%h -S'<name>' <from>..HEAD`). For the one or two most used pieces of shared code, call `cca:get_architecture`
with `module` set to the module holding it and say how many modules depend on it ("used by 6 modules"); skip a module newer than the scan.

**4. Work out the code move over the range.** With fewer than two trend scans in the range, or a named path (the trend measures the whole
repository), write in the Quality paragraph that there is no measured move; skip the rest. If the range covers every trend scan, the code
move is `codeMove.byTheCode` minus each unexplained move; else it is the last trend scan's `headline` minus the first's, minus the
`headlineDelta` of each mark and each unexplained move between them. Count the commits it covers: `git rev-list --count --no-merges <first
trend scan's commit>..HEAD`. Report each unexplained move to `cca:report_platform_issue` (`area` "scan-pipeline", `title` "Unexplained trend
move", both dates, commits and headlines in `description`); if that fails or did not load, keep the facts for the note after the changelog.

**5. Read what the scan says now.** Call `cca:audit` once and read only these fields. `scan` gives the commit and date for every statement
from the scan; if it is older than HEAD, say the later commits are not scanned. `lenses` and `plan` give each security dimension's dimension
score, `name` and `openInstances`; one absent from the lenses, or named in `gapReports` as not measured for a language, is not measured, so
its zero means nothing: say so. When `queue.resolutionsRefused` is above zero, findings marked resolved were found again: call
`cca:list_findings` (`state` `open`, walking `offset` until `truncated` is false) and write each finding with a `refusedResolution` as a
"did not hold" entry under Fixed, with its file; at zero, write nothing about it.

Call `cca:list_findings` (`dimension`, `state` `open`) only for a security dimension scoring below 10 or with `openInstances` above zero;
count `totalMatching` (for a named path, those under it on every page) and write one Still open entry with its first three places. With
none, write one "nothing open" entry naming the dimensions read. When commit messages say they work on findings in a dimension (duplication,
oversized classes, tests), call `cca:get_dimension` for at most the two most worked (the Quality paragraph holds no more); give its
`measured` values there in words ("duplicated lines now 1.2 % of the code"), with `peers` where given ("better than 80 % of repositories
scanned"). If a call fails, take Security from the git history alone; write in the Quality paragraph that open findings were not available.

**6. Write the changelog** with this template; omit empty sections, never the summary or Quality paragraph (three short sentences at most).

```markdown
## <from tag or commit> to <HEAD commit>[, <path> only] (<first date> to <last date>)

<Summary: N commits (plus M merges), <files> files. One sentence on the main thread of the work. Sources: every commit
message; diffs of the security, fix, revert and largest commits.>

### For maintainers
- **Use instead of writing again:** `<name>`, replacing <what>[, used by <n> modules] (`<commit>`)
- **Now enforced:** <rule>; `<test or lint rule>` fails when it breaks (`<commit>`)
- **Decided:** <ADR or approval, one sentence> (`<commit>`)
- **Kept on purpose:** <copies>; read the comment before merging one (`<commit>`)

### Added
### Changed
### Fixed
- <What was wrong and why; what it does now> (`<commit>`)
### Removed
- Reverted: <what>, because <reason> (`<commit>`)
### Security
- Fixed: `<package>` <old> to <new>, <advisory id> (`<commit>`)
- Still open: <dimension in words> (`<id>`) <count>, at <up to three places> (scan `<commit>`, <date>)

**Quality** (scan `<commit>`, <date>): <code move in points, over how many of the range's commits>. <Changes in how the
overall score is measured, as one total; each unexplained move, as not the code's.> <Measured values of the dimensions worked.>
```

**7. Check the changelog**, correct what fails, then check again: every requirement above holds; each cited hash exists and touches what its
entry names; every fix commit (`git log --no-merges -i --grep=fix --format='%h %s' <from>..HEAD`) is in an entry; counts, dates and files
match steps 1 and 2, Still open matches step 5, and the period reads the same everywhere. Where the repository's documentation (an ADR, a
README) disagrees with the code or the git history, the changelog says which is right.

**After the changelog, to the person who asked** (not part of the changelog), write a short paragraph on what the server gave this answer,
from the `contribution` items that changed it: each as what the changelog would otherwise have got wrong or lacked, with its number or file,
not repeating the Quality paragraph; if none changed it, leave it out. For an unexplained move not reported, add one sentence with the text
to send (both dates, commits, headlines). Write nothing before the changelog and nothing unrelated.

## Examples

For the maintainers, the default range (since `v2.3.0`):
> ## v2.3.0 to `e71b0c4` (3 Nov to 12 Nov)
> 41 commits (plus 6 merges), 112 files, mostly moving billing onto one invoice builder. Sources: every commit message;
> diffs of the security, fix, revert and five largest commits.
> ### For maintainers
> - **Use instead of writing again:** `Money.RoundOnce`, replacing rounding in four services, used by 5 modules (`b2c4e90`).
> - **Now enforced:** invoices are built only by `InvoiceBuilder`; `InvoiceArchTests` fails a direct `new Invoice` (`d81a7c2`).
> ### Fixed
> - Tax was rounded twice on multi-currency orders, once per currency; it is now rounded once at the total (`9f3e2a1`).
> ### Security
> - Fixed: `lodash` 4.17.20 to 4.17.21, GHSA-35jh-r3h4-6jhm (`4c7d0b8`).
> - Still open: security bugs (`D29`) 1, a SQL query built from user input at `src/reports/export.ts` (scan `e71b0c4`, 12 Nov).
>
> **Quality** (scan `e71b0c4`, 12 Nov): the code moved the overall score up 1.5 points, over the 33 commits after the first
> scan on 5 Nov. A change in how duplication is measured added 2 more, not from the code.

For a maintainer who names a path and a date ("what changed in billing since 1 November"):
> ## `a03d9f1` to `e71b0c4`, `src/billing/` only (1 Nov to 12 Nov)
> 7 commits, 14 files, moving credit notes out of the invoice engine. Sources: every commit message; diffs of the fix and
> largest commits.
> ### Changed
> - Credit notes are built by `CreditNoteBuilder` instead of inside `InvoiceEngine` (`b2c4e90`).
>
> **Quality** (scan `e71b0c4`, 12 Nov): the trend measures the whole repository, so `src/billing/` has no measured move; no
> security finding is open under it.
