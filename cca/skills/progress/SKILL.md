---
name: progress
description: "How this repository's quality has moved: what improved, what got worse, what changed in the measurement itself, and whether reported fixes held. Uses the cca MCP server's tools."
---

<!-- Generated from the same source as the cca server's MCP prompt of the same name; do not edit by hand. -->

How to read this service's data (apply to everything below):
- The scan is your map, not the whole answer. It has read every file at once, the git history and the vulnerability databases, the same way every run; you have the repository open. Use the scan to choose where to look, then open those files yourself and add what only reading shows: what the code is for, how it is designed, whether its names and tests say what it does, and whether a finding holds in its context. Keep apart, in what you write, what the scan measured (name its scan) and what you saw yourself (name the files you read).
- If reading shows a finding is wrong, say so and call dispute_finding with your evidence; never leave it out silently. A problem you find that the scan did not is yours to report as your own, and report_detector_gap sends it back.
- Start with audit, then get_summary. Name the scan they come from (its commit and date) beside every number you quote, and never mix numbers from two scans.
- get_summary's "summary" is written from templates whose every sentence is true wherever it can be printed: build on it and rephrase it for your reader, never contradict it. Use its "vocabulary" for what a dimension measures, what a kind of finding is and what to do about it; do not invent your own explanation of a check.
- A finding is what the scan found in the code, measured the same way on every repository. Never call a correct finding a matter of taste. Do not explain how the score is counted; say what was found and what it means.
- get_dimension's "peers" is a comparison, never part of the score. Many checks sit near the top for most repositories: quote the median beside a percentile, never a percentile alone.
- get_dimension's "reliability": when "measured" is false there are too few decided disputes to say how often this check is wrong; do not present its findings as certain, and do not call them noise either.
- Security findings first, then what moves the score most for the least work (the order audit's plan and next_task already give).
- Plain words, short sentences, in the language the person writes to you in. Say what was not measured as plainly as what was.
- End with the short line on what this service contributed, as the server's instructions say: only from the "contribution" items, never praise.

The task: report how this repository's quality has moved.
1. get_trend: the line on one ruler and its marks.
2. audit: the queue, especially "resolutionsRefused" (fixes an AI reported done that the scan found again) and "resolutionsImproved".
3. get_dimension for each lens or dimension that moved most between the first and the last point.
4. For the biggest moves, read the history behind them yourself (git log over those dates, for the files the findings name) so each move gets its cause in the code; for a fix that did not hold, open the file and say what is still there.
Write: the move in one sentence with its dates; what improved and why; what got worse and why; any change in how it is measured (a mark's sentence), kept apart from changes in the code; fixes that did not hold.
