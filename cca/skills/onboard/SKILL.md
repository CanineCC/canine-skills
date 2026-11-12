---
name: onboard
description: "An onboarding note for someone new to this codebase: what it is, how it is laid out, how work flows through it, where it changes most and where to be careful, from this service's map and the code it points to. Uses the cca MCP server's tools."
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

The task: write an onboarding note for a developer who is new to this codebase.
1. audit (its inventory: lines per language, tests) and get_summary (its words for what the scan holds).
2. get_architecture: the modules, their layers, which depend on which, and any cycles; for the busiest module, its blast radius.
3. list_findings with dimension "D15": the hotspots, files that change often and are complex; for the top three, get_architecture with "path" for the files that change together with them.
4. Open the entry points, the main files of the busiest module and the top hotspots, and read them: how a request or a job flows through the code, the conventions it follows, and what the names do not tell you. Read the README, the build files and the CI configuration for how to build, run and test it.
Write: what the codebase is and what it is made of; a map of its parts; how work flows through it; where it changes most and what changes together; where to be careful (hotspots, cycles, security findings); how to build, run and test it.
