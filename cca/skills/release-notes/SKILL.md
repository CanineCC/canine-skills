---
name: release-notes
description: "Writes release notes for the people who use the product: what is new, what changed, what was fixed, what security issues were closed or remain, and what they must do before upgrading. Use whenever someone asks for release notes, an update for customers or users, what a release or sprint gave customers, or a customer-facing changelog, in any form (a PDF, a document, an email). Uses the cca MCP server's tools."
---

# Release notes

## Words
- **users**: the people who use the product and do not write code. **operators**: the people who host or run it.
- **server**: the cca MCP server; its tools are named `cca:<tool>`. A **scan** is one run of it over the repository, named by
  its commit and date. A **finding** is one problem the scan found at a place in the code.
- **security dimensions**: the kinds of check whose findings are security findings: `D13` secrets in the code, `D28` secrets in
  the git history, `D29` security bugs found by static analysis, `D30` dependencies with known vulnerabilities, `D31` container
  and infrastructure configuration, `D32` personal data.
- **trend point**: one scan on the trend, with its `headline` (the quality score; the notes call it "the quality score"). A **mark** is a point on the trend where the
  way of measuring changed, with its own move `headlineDelta`. An **unexplained move**: two neighbouring trend points whose
  headline differs with no commit and no mark between them (an empty `git log <commit a>..<commit b>`); its size is the later
  headline minus the earlier.

## Rules for this skill
- **Signing in.** If the server's tools are missing or say you must sign in, stop before you write anything. Tell the person in
  one line: in Claude Code, type /mcp, choose cca and sign in (a server added during this chat may need a new chat). Write
  without the scan only if they ask, and then say at the top that the scan was not used.
- **Defaults.** Reader: users; operators get the "For operators" section. Range: from the last tag to HEAD, or,
  with no tag, over the last two weeks ("the last sprint" is the last two weeks unless a tag or date says otherwise). Form: a
  reply in the chat. The person's own words win over these defaults: the reader, the focus, the range and the form (a PDF or a
  document: make the file with what your tool has, or say it cannot). When something is missing, take the default, write the
  notes, and say in one line which default you used. Never hand the person a list of technical options to choose from.
- **Scan and reading.** Keep apart what the scan measured (name the scan by its date) and what you read in the commits and files. A security
  finding your reading shows is wrong: call `cca:dispute_finding` with your evidence and leave it out of the notes. Never explain how the score is counted; say what was found and what it means.
- **Numbers.** Every number comes from the server's answer or a count you made with git; a number names its scan or date.
- **Users.** The short version first. No tool names, file paths, commit hashes, dimension ids or internal names.

## Workflow

Copy this checklist and tick it off:

```
Release notes progress:
- [ ] 0. Server answers (Signing in)
- [ ] 1. Read the git history of the range
- [ ] 2. Security: fixed and still open
- [ ] 3. What the code did
- [ ] 4. Write the notes
- [ ] 5. Check the notes and add the closing paragraph
```

**0. Server answers (Signing in).** Call `cca:audit` (no arguments) and keep its `scan`: the commit and date the server last
scanned. If the tool is missing, fails or asks you to sign in: follow the sign-in rule.

**1. Read the git history of the range.** Read each commit message in `git log <last tag>..HEAD` (with no tag,
`git log --since="2 weeks ago"`; if git cannot read the history, tell the person in one line that the notes need the
repository's clone, and stop). When a merge message does not say what changed, open `gh pr view <n>`. When that does not say what users will notice
either, read the diff. Sort each change: new, improved, fixed, security, before you upgrade, for
operators, or internal. If the range holds no commit: the notes are one line saying nothing changed since the tag or date; go
to step 5.

**2. Security: fixed and still open.** Call `cca:list_findings` once for each security dimension, with that dimension and
`state` "open"; read `totalMatching` and walk the pages with `offset`. Sort each finding, testing in this order:
1. reading shows the finding is wrong: follow Scan and reading;
2. no commit after the step 0 scan touched its file (`git log <scan commit>..HEAD -- <file>` is empty, or the command fails
   because the scan commit is not in this clone): **open**;
3. a later change removes the problem: **fixed** (the next scan will confirm it); otherwise **open**.

A security change from step 1 with no finding (a dependency raised) is also fixed.
If `cca:list_findings` fails: write Security from step 1 alone, and tell the person outside the notes that open security findings
were not checked.

**3. What the code did.** Call `cca:get_trend` (no arguments) and take the trend points dated inside the range. What the
code did = the headline at the last of them minus the headline at the first, minus each mark's `headlineDelta` between them,
minus each unexplained move between them (report each with `cca:report_platform_issue`, `area` "scan-pipeline", with both
dates and headlines). Write it as one plain sentence under "For operators", with both dates and each mark's move apart. With
fewer than two trend points, a mark without `headlineDelta`, or a failed call: leave the sentence out.

**4. Write the notes** in this shape, in this order. The title takes the product's name from the README or package file
and the version from the tag (with no tag, the date range). Leave out a section with nothing in it, and every internal change:

```markdown
# <product> <version, or the date range>

<One sentence: the most important thing this release gives users.>

## New
<What users can do now, in their words.>

## Improved
<What works better.>

## Fixed
<What no longer goes wrong, with the date of the commit that fixed it.>

## Security
<What this release fixes. Each open item users should know: what is affected and what to do. Never how it could be used
against them.>

## Before you upgrade
<Anything users must do.>

## For operators
<Changes to hosting or running it, and what the code did from step 3 with its two dates.>
```

**5. Check the notes and add the closing paragraph.** Check, one by one:
- every fix you claim is a change you read in step 1;
- every security item called open or fixed was sorted by the step 2 rules against the step 0 scan;
- every open security item says what is affected and what to do, and none says how it could be used;
- every item is in the users' words, with no internal names, file paths, commit hashes or tool names;
- what the code did appears only under "For operators", with its dates, and no mark's move or unexplained move is told as
  the code's;
- the range you used is stated, and the form asked for was made (or you said it cannot be).

Fix each item that fails, then check the whole list again. When all pass, add the closing paragraph: a short paragraph on what
the server gave these notes, only from the `contribution` items that changed them, each said as what the notes would otherwise
have got wrong or lacked. If none changed them, leave it out.

## Examples

For users. Input, a pull request title: `fix(export): stream pdf pages past 100 rows`. Output, under "Fixed":

> Invoices with more than 100 lines now download as one PDF instead of stopping halfway (fixed 6 November).

Input, an open security finding on the sign-in page whose file no later commit touched. Output, under "Security":

> The sign-in page can still be loaded inside another website. Until the next release, sign in only at our own address.

For operators, under "For operators":

> The app now needs version 16 of its database; upgrade it before you install this release. Between the scans of 30 October
> and 13 November the code itself raised its quality score by 2 points; a change in how it is measured added 1 more.
