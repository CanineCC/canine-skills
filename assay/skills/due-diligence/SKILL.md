---
name: due-diligence
description: "Writes a due diligence memo on one codebase for the people deciding on a deal: what a buyer would inherit, the risks that carry a price, the questions for the seller, and what the seller should fix or disclose before closing, from its decision report and, where the code is at hand, a reading of it. Use whenever someone asks for due diligence on a codebase, what buying, investing in or taking over a piece of software would mean, or the technical risk in a deal. Uses the assay MCP server's tools."
---

# Due diligence memo

Default reader: the people deciding on the deal, who do not write code: follow "For a reader who does not write code" below.
Period: the newest finished report. Form: a memo in the chat. For a technical adviser to the buyer, file paths and dimension ids
are expected.

## Words
- **server**: the assay MCP server; its tools are named `assay:<tool>`.
- **codebase**: one repository the organisation can see, named owner/name.
- **evidence**: the signed scan results a supplier shared for a codebase. A **scan** is one run of the checks; the latest scan
  can be newer than the evidence.
- **report**: one decision report on a codebase, with its date, its verdict, its trend and its own text. Its **verdict** is its
  score, its band and the quality bar it was held against. **Verified**: the evidence signature checked out and the verdict
  reproduced from it; a report is verified or not, say which.
- **dimension**: one kind of check, with an id such as `D13`; a **lens** groups dimensions and has a key such as
  `architecture`. A **weak dimension** is a dimension or lens the report holds below its quality bar.

## Rules for this skill
- If the server's tools are missing, or ask for a key or a sign-in, stop before you write anything and say in one line that the
  assay server must be connected. Write without it only if the person asks you to, and then say at the top that the report was
  not used.
- The person's own words win over these defaults: the reader, the focus and the form (a PDF or a document: make it with what
  your tool has, or say it cannot). When something is missing, take the default and say in one line which you used. Never hand
  the person a list of technical options to choose from.
- Build on the report's text, never contradict it; rephrase it for the reader. Never explain how the score is counted; say what
  was found and what it means.
- Keep what the report measured (name it by its date, and whether it is verified) apart from what you read yourself (name the
  files). Without the code, say so and do not guess what is in it.
- Every number comes from a server answer or a count you made in the files; never estimate what you can count. An effort or
  cost you judge from reading is labelled "my estimate" and says what it rests on.

## Workflow

Copy this checklist and tick it off:

```
Due diligence progress:
- [ ] 0. Server answers
- [ ] 1. Find the codebase and its evidence
- [ ] 2. Read the newest report
- [ ] 3. Explain each weak dimension
- [ ] 4. Read the code, if it is at hand
- [ ] 5. Write the memo
- [ ] 6. Check the memo and add the closing paragraph
```

### 0. Server answers
Call `assay:list_codebases`, with `query` set to the name the person gave, if any. If the tool is missing or asks for a key or a
sign-in, follow the sign-in rule.

### 1. Find the codebase and its evidence
If none matches, say the codebase is not shared with the organisation and stop. If several match and the person named none, ask
which. Call `assay:get_codebase` with `repository` set to its owner/name as the list gives it. Keep its score from the evidence
and its score from the latest scan, who supplied the evidence, and its list of reports (id, date, status, score, band, quality
bar).

### 2. Read the newest report
The newest report is the one with the latest date whose `status` says it is finished. If none is finished, say so in one line and
stop. `assay:get_report` with `reportId` set to its id. Keep its date, the verdict, whether it is verified, the trend and the
report's own text.

### 3. Explain each weak dimension
For each weak dimension, `assay:explain_dimension` with `id` set to its dimension id or lens key. Keep what it affects in a deal
(price, delivery, risk) and who should care. If the call fails, explain it from the report's text and say so. If the report holds
nothing below its quality bar, "Risks that carry a price" says so in one line.

### 4. Read the code, if it is at hand
The report is the map: open what its weak dimensions point to (the modules, the dependency files, the configuration, the tests)
and read enough to say what fixing each would take. Count what can be counted (files that call a library, tests that cover a
module). A claim about what a piece of code does comes from reading it. Without the code, skip this step; "Not measured" says
the code was not read.

### 5. Write the memo
Use this shape:

```markdown
# Due diligence: <codebase>

<The verdict in three sentences, with the report's date, whether it is verified and who supplied the evidence.>

## What the buyer would inherit
<What holds and what does not, and which way the trend runs.>

## Risks that carry a price
<Ranked by cost. Each weak dimension: what it is, what it affects (price, delivery, security, people), who should care,
what fixing it would take, from the code where it was read.>

## Questions for the seller
<Each tied to something the report found.>

## Before closing
<What the seller should fix, and what they should disclose.>

## Not measured
<What the report did not cover; whether the code was read; when the latest scan's score differs from the evidence's,
both scores and that the evidence is older than the code.>
```

### 6. Check the memo and add the closing paragraph
Check the memo against the rules above and, for the default reader, the section below; also check that the verdict in the first
lines matches the report's, and that every weak dimension appears under "Risks that carry a price". Fix what fails and check
again until nothing fails. Then add the closing paragraph: a short paragraph on what the server gave this answer, only from the
`contribution` items that changed it, each as what the memo would otherwise have got wrong or lacked. If none changed it, leave
the paragraph out.

## For a reader who does not write code
- The short version first, on one screen, with the same numbers as the body.
- No tool names, check ids, commit hashes or score arithmetic; a lens or dimension only by what it measures, in a few words.
- Their language and their way of writing dates, one form throughout. Say what was not measured as plainly as what was.

## Examples

A line under "Risks that carry a price", for the people deciding on the deal:

> **Outdated payment library (security, price).** The verified report of 3 November finds the payment library two major versions
> behind, with known flaws. I read the checkout code: 14 files use the library, so upgrading it is weeks of work, my estimate from
> those 14 files. Ask the seller to upgrade before closing, or price it in.

The same line for the buyer's technical adviser:

> **D30, dependencies with known vulnerabilities.** The verified report of 3 November finds the payment library two major
> versions behind, with known flaws. It is called from 14 files under `app/payments/` (my count); the upgrade changes the charge
> API each of them uses, about three weeks, my estimate from that count.
