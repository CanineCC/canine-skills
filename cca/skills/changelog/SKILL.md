---
name: changelog
description: "Writes a changelog from the git history for the people who maintain the code, in Keep a Changelog sections with a security section and a line on how quality moved. Use whenever someone asks for a changelog, a list of changes, what changed or what was shipped since a date, tag or release. For customers or users, use release-notes. Uses the cca MCP server's tools."
---

# Changelog

**Defaults.** Reader: the people who maintain the code; commits, files and dimension ids stay in; for customers or users,
use the release-notes skill. Period: since the last tag, or from where the person names; with no tag, over the last two weeks.
Form: Markdown in the chat. When the person names a path, the changelog covers that **named path** only.
The person's own words win over these defaults: the reader, the focus, the period ("the last sprint" is the last two weeks
unless a tag or date says otherwise) and the form (a PDF or a document: make the file with what your tool has, or say it
cannot). When something is missing, take the default, do the work and say in one line which default you used; never hand the
person a list of technical options to choose from.

**Words.** The **server** is the cca MCP server (tools `cca:<tool>`). A **scan** is one run of it, named by commit and date. The
**headline** is the overall score; **score points** are points of it. A **trend point** is one scan on `cca:get_trend`. A
**mark** is an entry of the trend's `marks`: a change in the way of measuring, with its own move `headlineDelta` and its
`sentence`. An **unexplained move**: two neighbouring trend points whose `headline` differs with no commit and no mark between
them (an empty `git log <commit a>..<commit b>` and no mark). The **code move** is how much the code alone moved the headline.
**Security findings** are the open findings of `D13` (secrets in the code), `D28` (secrets in the git history), `D29` (security
bugs), `D30` (vulnerable dependencies), `D31` (container and infrastructure configuration) and `D32` (personal data).

## Rules for this skill

- **History and scan apart.** What changed and what was fixed come from the history (diff or message); what is still open comes
  from the scan, named by its commit and date.
- **Numbers.** Every number comes from the server's answer or a count you made with git.
- **Moves.** Never give a mark's move or an unexplained move as the code move. Report each unexplained move with
  `cca:report_platform_issue` (`area` "scan-pipeline", `title` "Unexplained trend move", both dates, commits and headlines in
  `description`); if that call fails, the Quality line says so.
- **Open findings.** A finding is measured the same way on every repository: never call a correct one a matter of taste.
  Never explain how the score is counted; say what was found and what it means.

## Workflow

Copy this checklist and tick it off:

```
Changelog progress:
- [ ] 0. Read the trend
- [ ] 1. Read the history
- [ ] 2. Code move over the range
- [ ] 3. Security now
- [ ] 4. Write the changelog
- [ ] 5. Check the changelog
```

**0. Read the trend.** `cca:get_trend`; keep the answer for step 2. If the server's tools are missing or say you must sign in,
stop before you write anything and tell the person in one line: in Claude Code, type /mcp, choose cca and sign in (a server added
during this chat may need a new chat). Write without the scan only if they ask, and then say at the top that it was not used. If
the call fails otherwise: skip step 2 and its Quality check, and the Quality line says the trend was not available.

**1. Read the history.** `git log --shortstat <from>..HEAD`, with `-- <path>` added for a named path. With no commits in the
range: say so in one line and stop. With 30 commits or fewer, read every diff; with more, read the diffs of the commits that
change more than 50 lines and take the rest from their messages. Sort the changes into added, changed, fixed, removed and
security, noting for each whether it comes from its diff or its message.

**2. Code move over the range.** With fewer than two trend points in the range, or a named path (the trend measures the whole
repository): the Quality line says there is no measured move; skip the rest of this step. Otherwise find each unexplained move
between the trend points in the range and report it (Rules). If the range covers every trend point, the code move is `codeMove`
minus each unexplained move. Otherwise it is the last trend point's `headline` minus the first one's, minus the `headlineDelta`
of each mark between them, minus each unexplained move. Keep each such mark's `sentence`.

**3. Security now.** `cca:audit`: keep its `scan` commit and date. Then `cca:list_findings` once per security dimension, each
with `dimension` set to it and `state` `open`. The count is `totalMatching`; for a named path, walk every page (`offset` until
`truncated` is false) and count only findings whose file is under the path. Write one Still open line per dimension with a count
above zero: its count and the first three places the answer lists. If the scan's commit is older than HEAD, the Still open lines
say the later commits are not yet scanned. If a call fails: write Security from the history alone, skip the Still open lines and
their check, and the Quality line says the open findings were not available.

**4. Write the changelog** in Keep a Changelog sections, under a range heading instead of a version. Group related commits into
one entry, one line each, citing the commit of the largest change in the group; leave out a section with nothing in it.

```markdown
## <from tag or commit> to <HEAD commit>[, <path> only] (<first date> to <last date>)

### Added
- <Entry> (`<commit>`)

### Changed
### Fixed
### Removed
### Security
- Fixed: <what> (`<commit>`)
- Still open: `<dimension>` <count> <what>, at <up to three places> (scan `<commit>`, <date>)

**Quality:** <The code move in score points; each mark's sentence; each unexplained move, as not the code's; the count of
security findings still open, with the scan.>
```

**5. Check the changelog.** Check: every entry traces to a commit in the range, from its diff or its message; each cited commit
exists; Security keeps fixed (history) apart from still open (scan); each Still open count matches step 3; the Quality line never
gives a mark's move or an unexplained move as the code move; every unexplained move was reported or the failed call is said;
nothing is stated that the history or the scan does not show. Fix what fails and check again.

**Closing paragraph.** End with a short paragraph on what the server gave this answer: only the `contribution` items that changed
it, each as what the changelog would otherwise have got wrong or lacked, with its number or file. If none changed it, leave it out.

## Examples

For the maintainers, the default range (since `v2.3.0`):

> ## v2.3.0 to `e71b0c4` (3 Nov to 12 Nov)
>
> ### Fixed
> - Invoices no longer round tax twice on multi-currency orders (`9f3e2a1`, with two follow-ups).
>
> ### Security
> - Fixed: raised `lodash` to 4.17.21, closing a known prototype-pollution flaw (`4c7d0b8`).
> - Still open: `D29` 1 SQL query built from user input, at `src/reports/export.ts` (scan `e71b0c4`, 12 Nov).
>
> **Quality:** the code moved the headline up 1.5 points over the range. A further 2 points come from a change in how
> duplication is measured on 8 Nov, not from the code. One security finding is still open (scan `e71b0c4`, 12 Nov).
>
> Without the scan, this changelog would have left out the open SQL query, which no commit in the range touched.

For a maintainer who names a path and a date ("what changed in billing since 1 November"):

> ## `a03d9f1` to `e71b0c4`, `src/billing/` only (1 Nov to 12 Nov)
>
> ### Changed
> - Credit notes are built by `CreditNoteBuilder` instead of inside `InvoiceEngine` (`b2c4e90`).
>
> **Quality:** the trend measures the whole repository, so `src/billing/` has no measured move; no security finding is open
> under it (scan `e71b0c4`, 12 Nov).
