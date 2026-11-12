---
name: onboard
description: "An onboarding note for someone new to this codebase: what it is, how it is laid out, where it changes most and where to be careful. Uses the cca MCP server's tools."
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
The task: write an onboarding note for a developer who is new to this codebase.
1. audit (its inventory: lines per language, tests) and get_summary (its words for what the scan holds).
2. get_architecture: the modules, their layers, which depend on which, and any cycles; for the busiest module, its blast radius.
3. list_findings with dimension "D15": the hotspots, files that change often and are complex; for the top three, get_architecture with "path" for the files that change together with them.
Write: what the codebase is and what it is made of; a map of its parts; where it changes most and what changes together; where to be careful (hotspots, cycles, security findings); how it is tested.
