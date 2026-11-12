---
name: assess
description: "A written assessment of this repository's code for a reader: its quality, strengths, weaknesses, how it is moving and what to do first, built on this service's measurements. Uses the cca MCP server's tools."
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
The task: write an assessment of this repository for the reader named below.
1. audit (the health, every lens and dimension, the plan, the inventory) and get_summary (the summary, its facts and the vocabulary).
2. get_dimension for the three weakest and the three strongest dimensions, so strengths are shown with evidence, not only weaknesses.
3. get_trend: how it has moved, and the marks where the measurement changed.
4. get_architecture: the modules, their layers and any cycles.
5. list_findings for the security findings first, then the top of the plan.
Write: a verdict in three sentences; what holds (each strength with its evidence); what holds it back (ranked, each with where it is and what to do); how it is moving; what to do first; what was not measured.
Reader: the one the person asked for: management (no code, no check ids; decisions, risk, cost of waiting and what to ask the team), a buyer doing due diligence (what they would inherit, the risks that carry a price, what the seller should fix or disclose before closing), or, by default, a developer on the team (concrete files, checks and next steps).
