---
name: retrospective
description: "A retrospective of the code's quality over a sprint: what went well and what did not, with evidence from the scan and the git history, what changed in how it is measured kept apart, and what to try next. Uses the cca MCP server's tools."
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
- Never show the person tool names, fingerprints or check ids: say in plain words what you did or would do, and why. Keep it to what they can act on: the short version fits one screen; file lists and commit trails go at the end, briefly, or are left out.
- Every number you give comes from an answer of this service or a count you made. Count with git and the files; never estimate a number you could count. A claim about what a test or a function does comes from reading what it actually calls; if you did not check, say so.
- End with the short closing paragraph the server's instructions ask for: what the service gave this answer, as its worth to the person (what the answer would otherwise have got wrong or lacked), only from the "contribution" items that changed it, never praise.

The task: a retrospective of the code's quality over the last two weeks, or since the date or tag the person names, for the team's meeting.
1. get_trend: the points in the period and the marks inside it. What the code did over the period is the headline at the last point in it minus the first, apart from each mark's move inside it; say that number.
2. audit: resolutionsRefused (fixes reported done that did not hold), resolutionsImproved, and the queue.
3. git log over the period yourself: what was worked on, by area; read the commits behind the biggest move in a lens, up or down.
4. list_findings for the security findings, and get_dimension for the dimension that moved most: what arrived and what left.
Write: what went well (each with its evidence: a move, a fix that held, a finding gone); what did not (a fix that did not hold, a finding that arrived, a lens that fell, each tied to the commits behind it); what changed in how it is measured, kept apart; three things to try next sprint, each concrete and checkable by the next scan. No blame on people: say what happened to the code.
