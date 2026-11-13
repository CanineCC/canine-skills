---
name: review-branch
description: "Reviews a branch or pull request before it is merged: the findings it would add and remove in a scan preview, new dependencies between modules, and a reading of the diff, ending in a verdict. Use whenever someone asks to review a branch, a PR, a merge request or their changes before merging. Uses the cca MCP server's tools."
---

# Branch review

Default reader: the branch's author, who writes code, so file paths, lines and commits belong in the review. For a reader who
does not write code, follow "For a reader who does not write code" below and write the verdict in plain words. Period: the
branch against its base. Form: a review in the chat. Branch: the one the person names, or else the one checked out. Base: the
default branch (`git symbolic-ref refs/remotes/origin/HEAD`), unless the person names another.

## Words
- **server**: the cca MCP server; its tools are named `cca:<tool>`. A **scan** is one run of it, named by its commit and date.
- **finding**: one problem the scan found at a file and line, with a `fingerprint` and a **dimension** (a kind of check, with
  an id such as `D15`). The **headline** is the overall score.
- **arriving** finding: one the preview adds; **leaving** finding: one it removes.
- **moved** finding: a leaving finding whose problem appears again as an arriving finding elsewhere in the diff.
- **hidden** finding: one that leaves because the code was silenced (a suppression comment, an excluded path), not fixed.
- **security finding**: a finding in `D13` secrets in the code, `D28` secrets in the git history, `D29` security bugs,
  `D30` dependencies with known vulnerabilities, `D31` container and infrastructure configuration or `D32` personal data.
- **drift**: a new dependency that `cca:check_dependencies` judges "closes-cycle" or "points-outward"; a **new-dependency** is
  any other new dependency between modules it reports.

## Rules for this skill
- If the server's tools are missing or say you must sign in, stop before you write anything. Tell the person in one line: in
  Claude Code, type /mcp, choose cca and sign in (a server added during this chat may need a new chat). Write without the scan
  only if they ask, and then say at the top that the scan was not used.
- The person's own words win over these defaults: the reader, the base, the focus and the form (a PDF or a document: make it
  with what your tool has, or say it cannot). When something is missing, take the default and say in one line which you used.
  Never hand the person a list of technical options to choose from.
- Every number comes from the preview, another server answer or a count you made with git; each score names its scan.
- Keep what the scan measured (name the scan) apart from what you read in the diff (name the files).
- A finding is measured the same way on every repository: never call a correct one a matter of taste.
  Never explain how the score is counted; say what was found and what it means.

## Workflow

Copy this checklist and tick it off:

```
Branch review progress:
- [ ] 0. Start the preview
- [ ] 1. Scan preview of the branch
- [ ] 2. Read the diff
- [ ] 3. New dependencies between modules
- [ ] 4. Tie each arriving and leaving finding to the diff
- [ ] 5. Write the review
- [ ] 6. Check the review and add the closing paragraph
```

### 0. Start the preview
Call `cca:request_rescan` with `branch` set to the branch, as git names it; this starts the preview that step 1 polls. If the
tool is missing or asks you to sign in, follow the sign-in rule. If it answers "Disabled" or "RateLimited": there is no preview
(see step 1).

### 1. Scan preview of the branch
Call `cca:get_branch_preview` with `branch` once a minute while `status` is Queued or Running, for at most 30 minutes. When it
is Done, keep the scan it compares against (commit and date), the arriving and leaving findings (each with `fingerprint`,
dimension, file and line), each dimension's score then and now, and the headline then and now.
If there is no preview (Failed, NotFound, not Done in 30 minutes, or step 0 said so): skip step 4, the preview parts of step 5
and the arriving, leaving and score checks of step 6; say so at the top of the review.

### 2. Read the diff
Read the diff from the merge base to the branch, file by file.

### 3. New dependencies between modules
Call `cca:get_architecture` with no arguments and take `baseline.commit`, the commit the module graph was measured at. Pass
`git diff -U100000 <baseline.commit>..<branch>` as `diff` to `cca:check_dependencies` (the wide context puts each changed file in
whole, so its imports are seen). Keep only the drifts and new-dependencies that come from lines the branch adds (the step 2
diff); drop those from changes already on the base.
If either call fails: say at the top of the review that dependencies between modules were not checked, and skip the drift and
new-dependency check of step 6.

### 4. Tie each arriving and leaving finding to the diff
For each arriving finding, find the diff lines that cause it. If the diff shows it is wrong, call `cca:dispute_finding` with its
`fingerprint` and your evidence as `reason`; it then goes under "Remarks" only, never under "Must change". If the dispute call
fails: list the finding under "Remarks" as not filed.
For each leaving finding, check in the diff what happened to its problem:
- fixed: it goes under "What the branch improves";
- moved: pair it with its arriving finding; the problem is listed once, as that arriving finding, at its new file and line,
  with the words "moved from `<old file>`";
- hidden: it goes under "Must change", at the file and line that silences it.

### 5. Write the review
Use this shape:

```markdown
**Verdict:** <merge | merge after changes | not this way>, <one line why>

## Must change
- `<file>:<line>`: <what and why>

## What the branch improves
<Headline then and now; each dimension whose score moves, then and now; the leaving findings checked as fixed
in step 4. All from the preview, naming its scan. Without a preview: only what the diff visibly fixes, labelled as your reading.>

## Remarks
<Your own reading: naming, design, tests; each new-dependency; each disputed finding. Kept apart from what the scan measured.>
```

"Must change" holds, in this order: arriving security findings, every drift, the other arriving findings that are not disputed
(a moved finding among them, once), and every hidden finding. The verdict is "merge" when "Must change" is empty, "not this way"
when an item there cannot be fixed without redoing the branch's approach, and "merge after changes" otherwise.

### 6. Check the review and add the closing paragraph
Check, one by one:
- every "Must change" item names a file and a line, in the step 2 diff;
- every arriving finding is under "Must change" or disputed under "Remarks"; every leaving one was checked as fixed, paired
  with its arriving finding as moved, or is under "Must change" as hidden;
- no problem is listed twice (a moved finding appears once, at its new place);
- every drift is under "Must change" and every new-dependency under "Remarks";
- every score comes from the preview and names its scan, and your own reading is kept apart from it;
- the verdict follows the rule in step 5;
- a missing preview or dependency check is said at the top;
- for a reader who does not write code, the review passes the section below.

Fix each item that fails, then check the whole list again. When all pass, add the closing paragraph: a short paragraph on what
the server gave this answer, only from the `contribution` items that changed it, each as what the review would otherwise have
got wrong or lacked. If none changed it, leave the paragraph out.

## For a reader who does not write code
The verdict in plain words first, on one screen, with the same numbers as the body. No tool names, fingerprints, commit hashes
or dimension ids; a dimension only by what it measures, in a few words; the scan by its date. Say what was not checked as plainly
as what was.

## Examples

For the branch's author, two "Must change" items and a line under "Remarks":

> - `src/api/orders.ts:88`: the new handler builds its SQL from the request body, and the preview flags it as an injection
>   risk; pass the values as query parameters instead.
> - `src/report/export.ts:12`: imports `src/api/`, which closes a cycle between report and api; move the shared type to
>   `src/shared/`.
>
> Under "Remarks": disputed the preview's finding that `src/util/retry.ts:40` swallows errors; line 44 rethrows after logging.

For a teammate deciding whether to merge, the verdict and "What the branch improves":

> **Verdict:** merge after changes, one new SQL injection risk in `src/api/orders.ts`; the rest is a clear gain.
>
> Against the scan of `main` at `3f2a91c` (10 November), the preview moves the headline from 71 to 73, and the duplicated-code
> score rises from 6.2 to 7.8 out of 10 (higher means less duplication): the 14 copies of the date parser in `src/report/`
> become one module.
