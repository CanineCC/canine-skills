---
name: review-branch
description: "Review a branch before it is merged: the scan of its head against the default branch (what findings arrive and leave, which scores move), and your own reading of the diff. Uses the cca MCP server's tools."
---

<!-- Generated from the same source as the cca server's MCP prompt of the same name; do not edit by hand. -->

How to read this service's data (apply to everything below):
- The scan is your map, not the whole answer. It has read every file at once, the git history and the vulnerability databases, the same way every run; you have the repository open. Use the scan to choose where to look, then open those files yourself and add what only reading shows: what the code is for, how it is designed, whether its names and tests say what it does, and whether a finding holds in its context. Keep apart, in what you write, what the scan measured (name its scan) and what you saw yourself (name the files you read).
- If reading shows a finding is wrong, say so and call dispute_finding with your evidence; never leave it out silently. A problem you find that the scan did not is yours to report as your own, and report_detector_gap sends it back.
- Start with audit, then get_summary. Name the scan they come from (its commit and date) beside every number you quote, and never mix numbers from two scans.
- get_summary's "summary" is written from templates whose every sentence is true wherever it can be printed: build on it and rephrase it for your reader, never contradict it. Use its "vocabulary" for what a dimension measures, what a kind of finding is and what to do about it; do not invent your own explanation of a check.
- A finding is what the scan found in the code, measured the same way on every repository. Never call a correct finding a matter of taste. Do not explain how the score is counted; say what was found and what it means.
- get_trend: a score on the line is the score as it was measured then. A move across a mark is partly or wholly a change in how it is measured, not in the code. Say what the code did with "codeMove", and each mark's sentence for the rest; never present a mark's move as the code's.
- get_dimension's "peers" is a comparison, never part of the score. Many checks sit near the top for most repositories: quote the median beside a percentile, never a percentile alone.
- get_dimension's "reliability": when "measured" is false there are too few decided disputes to say how often this check is wrong; do not present its findings as certain, and do not call them noise either.
- Security findings first, then what moves the score most for the least work (the order audit's plan and next_task already give).
- Plain words, short sentences, in the language the person writes to you in. Say what was not measured as plainly as what was.
- End with the short closing paragraph the server's instructions ask for: what the service gave this answer, as its worth to the person (what the answer would otherwise have got wrong or lacked), only from the "contribution" items that changed it, never praise.

The task: review the branch the person names, or else the branch checked out before it is merged.
1. request_rescan with branch set to it, then get_branch_preview once a minute until it is Done: the findings that would arrive and leave, and each dimension's score then and now. A preview changes nothing on the project.
2. Read the change yourself: git diff from where the branch left the default branch to its head, file by file.
3. check_dependencies with that diff (`git diff -U100000` against the baseline commit get_architecture gives): the new dependencies between modules and whether any closes a cycle.
4. For each finding that would arrive, find the lines in the diff that cause it. For each that would leave, check in the diff that it was fixed, not moved or hidden.
Write the review as a reviewer would: a verdict in one line (merge, merge after changes, or not this way); what must change before merging, each tied to a file and line; what the branch improves, with the findings it removes and the scores that move; then your own remarks on the diff (naming, design, tests), kept apart from what the scan measured.
