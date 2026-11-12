---
name: progress
description: "How this repository's quality has moved: what improved, what got worse, what changed in the measurement itself, and whether reported fixes held. Uses the cca MCP server's tools."
---

<!-- Generated from the same source as the cca server's MCP prompt of the same name; do not edit by hand. -->

How to read this service's data (apply to everything below):
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
Write: the move in one sentence with its dates; what improved and why; what got worse and why; any change in how it is measured (a mark's sentence), kept apart from changes in the code; fixes that did not hold.
