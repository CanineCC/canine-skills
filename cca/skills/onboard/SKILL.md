---
name: onboard
description: "Writes an onboarding note for someone new to a codebase: what it is, its open security findings, how it is laid out, how work flows through it, where it changes most, where to be careful, and how to build, run and test it. Use whenever someone asks to onboard a person, write an onboarding note, a getting-started or welcome guide, an introduction or a 'how this code works' for a new developer or colleague. Uses the cca MCP server's tools."
---

# Onboarding note

Default reader: a developer joining the team; file paths, function names and commit ids are expected. Period: the whole git
history the scan read. Form: a Markdown note in the chat. If the reader does not write code (a designer, a manager), follow
the rule for that reader below.

## Words
- **server**: the cca MCP server; its tools are named `cca:<tool>`. A **scan** is one run of it over the repository, named by
  its commit and date. A **finding** is one problem the scan found at a place in the code.
- **dimension**: one kind of check, with an id such as `D15`. The **security dimensions** are `D13` secrets in the code, `D28`
  secrets in the git history, `D29` security bugs found by static analysis, `D30` dependencies with known vulnerabilities,
  `D31` container and infrastructure configuration, `D32` personal data.
- **hotspot**: a file with an open `D15` finding (changes often and is complex); its **co-change partners** are the files that
  changed together with it in the git history. The **busiest module** is the module the most other modules use.

## Rules for this skill
- **Signing in.** If the server's tools are missing or say you must sign in, stop before you write anything. Tell the person in
  one line: in Claude Code, type /mcp, choose cca and sign in (a server added during this chat may need a new chat). Write
  without the scan only if they ask, and then say at the top that the scan was not used.
- **Defaults.** The person's own words win over these defaults: the reader, the focus and the form (a PDF or a document: make
  the file with what your tool has, or say it cannot). When something is missing, take the default, write the note, and say
  in one line which default you used. Never hand the person a list of technical options to choose from.
- **Scan and reading.** The scan says where to look; your reading says what the code is for and whether a finding holds. Keep
  the two apart: name the scan for what it measured, the files for what you saw. A finding your reading shows is wrong: say
  so and call `cca:dispute_finding` with your evidence. A problem the scan missed: give it as your own and report it with
  `cca:report_detector_gap`. Never call a correct finding a matter of taste. Never explain how the score is counted; say what
  was found and what it means.
- **Summary.** `summary` from `cca:get_summary` is true wherever it is printed: build on it, never contradict it.
- **Numbers.** Every number comes from the server's answer or a count you made with git or the files, and names its scan. A
  claim about what code does comes from reading it; if you did not read it, say so.
- **A reader who does not write code.** The short version first, on one screen, with the same numbers as the body. No tool
  names, commit hashes or dimension ids; a dimension only by what it measures, in a few words.

## Workflow

Copy this checklist and tick it off:

```
Onboarding progress:
- [ ] 0. Server answers (Signing in)
- [ ] 1. Inventory and summary
- [ ] 2. Open security findings
- [ ] 3. Map of the parts
- [ ] 4. Hotspots and their co-change partners
- [ ] 5. Read the code and the build
- [ ] 6. Write the note
- [ ] 7. Check the note and add the closing paragraph
```

### 0. Server answers (Signing in)
Call `cca:audit` (no arguments). If the tool is missing or says you must sign in, follow the sign-in rule.

### 1. Inventory and summary
From the `cca:audit` answer keep `scan` (its commit and `scannedAt`) and `inventory` (the lines the scan read, per language).
Then `cca:get_summary`: keep `summary` for the opening, and use its `vocabulary` for any dimension the note names, instead of
your own explanation.

### 2. Open security findings
`cca:list_findings` once for each security dimension, each with `dimension` set to its id and `state` "open". Keep
`totalMatching` and, for each row, its file and what it found. A newcomer must know these before touching the code, so they get
their own section right after the opening. If every count is zero, that section is one line: the scan found none open.

### 3. Map of the parts
`cca:get_architecture` with no arguments: each module with its layer, level, how many modules use it, and the cycles. Call it
again with `module` set to the busiest module (on a tie, the first listed): keep its blast radius (every module that depends on
it, directly or not). If the first answer lists no modules, build "The parts" from the top-level directories and say in one line
that the scan has no module map for this codebase.

### 4. Hotspots and their co-change partners
`cca:list_findings` with `dimension` "D15" and `state` "open". The hotspots are the first three distinct files in the order the
rows come back (most severe first). Fewer than three: use those and say so in one line. None: write that the scan found no file
that is both changed often and complex, and skip the rest of this step. For each hotspot, `cca:get_architecture` with `path`
set to it: keep its co-change partners and how often each changed with it; if none is named, quote the answer's `note`.

### 5. Read the code and the build
Open the entry points, the main files of the busiest module and the hotspots: how one request or job flows, the conventions the
code follows, what the names do not tell you. Count the test files (`git ls-files` matching the test folders or names the build
uses). Read the README, the build files and the CI configuration for the exact commands to build, run and test.

### 6. Write the note
Use this shape; adapt the headings to the codebase:

```markdown
# Welcome to <project>

<Three sentences: what it is, what it is built with (lines per language from the scan of <date>), the one thing to know first.>

## Open security findings
<Each open finding: file and what it means, with the scan's date. Or: none open in the scan of <date>.>

## The parts
<Each module in a line: what it does, what it depends on.>

## How work flows
<One request or job from entry point to result, naming the files.>

## Where it changes most
<Each hotspot and its co-change partners, with how often they changed together.>

## Where to be careful
<The busiest module and its blast radius, cycles, conventions that bite.>

## Build, run and test
<The exact commands from the README, build files and CI; how many test files there are (my count).>

## First week
<Three to five concrete first steps.>
```

### 7. Check the note and add the closing paragraph
Check the note against the rules above, and also: every file you name exists; every build command matches the build files or
CI; the security counts match `totalMatching`; for a reader who does not write code, the note passes that rule. Fix what fails
and check again until nothing fails.

Then add the closing paragraph: a short paragraph on what the server gave this note, only from the `contribution` items that
changed it, each said as what the note would otherwise have got wrong or lacked. If none changed it, leave it out.

## Examples

A line under "Where it changes most", for a developer:

> `src/billing/invoice.ts` changed in 41 commits (scan of 12 November, commit `a1b2c3d`) and is the most complex file in the
> billing module; `src/billing/tax.ts` changed together with it in 30 of those commits, so change them together.

The same line for a designer joining the team:

> Billing changes more often than any other part of the product: the invoice code changed 41 times in its history (scan of
> 12 November), and the tax rules changed with it in 30 of those 41. Expect work on invoices to touch tax as well.
