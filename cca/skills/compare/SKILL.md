---
name: compare
description: "Reads ten production files closely on its own, then compares that review with the repository's code-quality scan of the same files in three piles (found by both, found only by the scan, found only by the review), disputing wrong findings and reporting missed problems. Use when someone asks for a manual or AI code review checked against the scan, or what an AI review misses compared with a full scan. Uses the cca MCP server's tools."
---

# Review against the scan

**Defaults.** Reader: the person who asked, who writes code. Period: the latest scan. Form: Markdown in the chat. Files: 10
production files chosen as in step 1. The person's own words win over these defaults: the reader, the files or their number, and
the form, such as a PDF. When something is missing, take the
default, do the work and say in one line which default you used; never hand the person a list of technical options to choose from.

**Words.** The **server** is the cca MCP server (tools `cca:<tool>`). A **scan** is the server's analysis of the repository at one commit, named by commit and date. A
**finding** is one problem the scan found at one place. A **dimension** is the kind of problem a finding belongs to, named by
an id such as `D15`. A **production file** ships (not a test, fixture, generated or vendored
file). **The review** is your own reading in step 2; each problem it lists is a **review item**. A
**fingerprint** is a finding's stable id, as `cca:list_findings` returns it; a **receipt** is the id `cca:report_detector_gap`
returns. The **`contribution`** items are the facts each answer of the server says it added.

## Rules for this skill

- **The review stays your own.** Apart from the hotspot paths in step 0, nothing the scan says about the 10 files is read before the review is
  written.
- **Numbers.** Every number comes from the server's answer or a count you made (`wc -l`, counting the piles); never estimate.
- **Findings.** A finding is measured the same way on every repository: never call a correct one a matter of taste.

## Workflow

Copy this checklist and tick it off:

```
Compare progress:
- [ ] 0. Hotspot paths
- [ ] 1. Choose 10 files
- [ ] 2. Your own review
- [ ] 3. The scan's findings in those files
- [ ] 4. Sort into three piles and follow up
- [ ] 5. Write the report
- [ ] 6. Check the report and add the closing paragraph
```

**0. Hotspot paths.** `cca:list_findings` with `dimension` `D15` (hotspots: files that change often and are complex). It is the
only call to the server before step 3. Note only the file path of each row, in the order returned, before reading anything else
from the answer and set the rest aside. If the server's tools are missing or say you must sign in,
stop before you write anything and tell the person in one line: in Claude Code, type /mcp, choose cca and sign in (a server
added during this chat may need a new chat). Write without the scan only if they ask, and then say at the top that it was not used.

**1. Choose 10 files.** Keep the first 10 distinct production files among the noted paths. If fewer, fill up with the largest
production files from:

```bash
git ls-files -z | xargs -0 wc -l | grep -v ' total$' | sort -rn
```

If git fails (not a git checkout), fill up with the largest production files you can list, and say so.

**2. Your own review.** Read the 10 files in full and list review items: defects, security problems, duplication,
complexity and dead code, each with file and line. Write the list into the conversation now; it does not change once you see
the scan.

**3. The scan's findings in those files.** `cca:audit`: keep `scan` (commit and date) and `inventory.productionLines`, the production lines the scan read in the whole repository.
Then `cca:list_findings` with no filter, walking every page with `offset` until `totalMatching` is reached, and keep the findings
whose file is one of the 10. If none are: both scan piles are empty; say so and go on.

**4. Sort into three piles and follow up.** Found by both; found only by the scan; found only by the review. A review item and a
finding are the same problem when they name the same file, lie within 5 lines of each other and describe the same problem;
otherwise they go in separate piles.
- Found only by the scan: `cca:get_finding` with its fingerprint, then read the cited lines and say whether it holds. One that
  does not: `cca:dispute_finding` with its fingerprint and your evidence, naming what in the code the scan could have checked.
- Found only by the review: take a second look. Each review item you still believe: `cca:report_detector_gap` with `dimension`
  (the dimension id of a scan finding of the same kind; with none, a few words naming the problem), `title`, `file` and your evidence as
  `description`, and keep the receipt it returns. If the call fails: the review item is listed as not reported, with the reason.

**5. Write the report** in this shape:

```markdown
# Review against the scan: <project>

Files read: <n> files, <lines> lines (counted with wc -l). Production lines the scan read in the repository: <inventory.productionLines>, scan <commit>, <date>.

## Found by both (<count>)
<Each: file, line, what.>
## Found only by the scan (<count>)
<Each: holds, or disputed with the reason.>
## Found only by the review (<count>)
<Each with its receipt, or "not reported" and why.>

## What this shows
<What the scan found that the review missed, and what the review found that the scan missed, without softening either.>
```

**For a reader who does not write code:** the short version first, with the same numbers as the body; no tool names,
fingerprints, commit hashes or dimension ids; write a receipt as "reported, reference kept"; say what was not covered as plainly
as what was.

**6. Check the report and add the closing paragraph.** Check: each count matches its list; the review was written before step 3; every disputed finding has its
call and every reported review item its receipt; the lines read are counted, not estimated; nothing in "What this shows" is
softer than the piles; a disputed finding counts in no total of problems. Fix what fails and check again. Then end with a short paragraph on what the server gave this answer: only the `contribution` items that changed
it, each as what the report would otherwise have got wrong or lacked, with its number or file. If none changed it, leave it out.

## Examples

For the developer who asked:

> **Found only by the scan.** `src/cache/store.go`: a lock is taken on line 88 and not released on the error path at line 97.
> Holds; the review missed it because it followed only the happy path. `src/api/auth.go` line 52, "hard-coded secret": disputed;
> the string is the name of an environment variable, not its value, and the scan could have checked the `os.Getenv` call around it.
>
> **Found only by the review.** `src/api/orders.go` line 140: the page size from the query string is never capped. Reported,
> receipt `gap-3f19`. `src/api/orders.go` line 211: an error from `tx.Rollback` is dropped. Not reported: the call failed
> (server timeout).
>
> **What this shows.** In 10 files (3,412 lines, counted with wc -l) the scan found 14 findings (scan `8d2e41b`, 12 Nov) and the
> review found 9 review items; 6 are in both. 8 were found only by the scan, 1 of them disputed, among them concurrency and history problems a
> single reading cannot see. The review found 3 the scan missed, among them an input check it has no rule for; 2 are reported.
>
> Without the scan, this review would have missed the lock left held on line 97 of `src/cache/store.go`.

For a manager who does not write code:

> **What this shows.** In ten of the busiest files, the scan found 13 problems that hold (and 1 more that is disputed) and the review found 9; 6 were found by both
> (scan of 12 November). 7 were found only by the scan, among them a lock left held when an error occurs. The review caught
> 3 the scan missed; 2 were reported, reference kept, and 1 could not be reported because the server did not answer.
>
> Without the scan, the review alone would have missed 7 of the 16 problems that hold in these files (the disputed one left
> out).
