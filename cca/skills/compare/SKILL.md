---
name: compare
description: "Review a few files yourself, then compare your review with what the scan found. Use when someone asks you to review specific files and check them against the scan, or asks what an AI review misses compared with a full scan. Uses the cca MCP server's tools."
---

<!-- Generated from the same source as the cca server's MCP prompt of the same name; do not edit by hand. -->

How to read this service's data (apply to everything below):
- The scan is your map, not the whole answer. It has read every file at once, the git history and the vulnerability databases, the same way every run; you have the repository open. Use the scan to choose where to look, then open those files yourself and add what only reading shows: what the code is for, how it is designed, whether its names and tests say what it does, and whether a finding holds in its context. Keep apart, in what you write, what the scan measured (name its scan) and what you saw yourself (name the files you read).
- If reading shows a finding is wrong, say so and call dispute_finding with your evidence; never leave it out silently. A problem you find that the scan did not is yours to report as your own, and report_detector_gap sends it back.
- Start with audit, then get_summary. Name the scan (its commit and date) beside every number about the code now. A trend point is named by its date. Compare numbers from two scans only along the trend.
- get_summary's "summary" is written from templates whose every sentence is true wherever it can be printed: build on it and rephrase it for your reader, never contradict it. Use its "vocabulary" for what a dimension measures, what a kind of finding is and what to do about it; do not invent your own explanation of a check.
- A finding is what the scan found in the code, measured the same way on every repository. Never call a correct finding a matter of taste. Do not explain how the score is counted; say what was found and what it means.
- get_trend: a score on the line is the score as it was measured then. A move across a mark is partly or wholly a change in how it is measured, not in the code. Say what the code did with "codeMove", and each mark's sentence for the rest; never present a mark's move as the code's. A move between two points with no commit between them in git and no mark is not the code's either: say it is unexplained, keep it out of what the code did, and report it with report_platform_issue.
- get_dimension's "peers" is a comparison, never part of the score. Many checks sit near the top for most repositories: quote the median beside a percentile, never a percentile alone.
- get_dimension's "reliability": when "measured" is false there are too few decided disputes to say how often this check is wrong; do not present its findings as certain, and do not call them noise either.
- Security findings first, then what moves the score most for the least work (the order audit's plan and next_task already give).
- Write for the person who asked. Unless they show they write code, assume they may not: the short version first, plain words, and no check id or lens name without saying in a few words what it means. Short sentences, in the language they write to you in, with dates and times as they would say them. The short version uses the same numbers as the body. Say each thing once. Say what was not measured as plainly as what was.
- The person's own words win over this recipe's defaults: the reader, the focus (for example what the work gave their customers), the period ("the last sprint": the last two weeks unless a tag or date says otherwise; say which range you used) and the form (a PDF or a document: write it, then make the file with what your tool has, or say plainly that it cannot).
- Ask only what you cannot find out or sensibly assume. When something is missing (a starting point, a reader, a capacity), take the default the recipe gives, do the work, and say in one line which you used and that they can name another. Never hand the person a list of technical options to choose from.
- Never show the person tool names, fingerprints or check ids: say in plain words what you did or would do, and why. Keep it to what they can act on: the short version fits one screen; file lists and commit trails go at the end, briefly, or are left out.
- Every number you give comes from an answer of this service or a count you made. Count with git and the files; never estimate a number you could count. A claim about what a test or a function does comes from reading what it actually calls; if you did not check, say so.
- End with the short closing paragraph the server's instructions ask for: what the service gave this answer, as its worth to the person (what the answer would otherwise have got wrong or lacked), only from the "contribution" items that changed it, never praise.

The task: an honest comparison between your own review and this service's scan, on the same files.
1. Choose 10 production files: the top hotspots (list_findings with dimension "D15"), then the largest files.
2. BEFORE you call any other tool of this service, review those files yourself for defects, security problems, duplication, complexity and dead code. Write your list down.
3. Then list_findings, and keep the findings in those files.
4. Sort everything into three piles: found by both; found only by the scan; found only by you. For each item found only by the scan, open it and say whether it holds; one that does not, dispute_finding with your evidence. For each item in the last pile you still believe after a second look, call report_detector_gap, and keep its receipt.
5. Report the three piles with their counts, the files you read and how many lines they hold beside the lines the scan read (audit's inventory), and the receipts of what you reported. Say what the scan found that you missed, and what you found that it missed, without softening either.
