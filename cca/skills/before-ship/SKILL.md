---
name: before-ship
description: "Before you ship it or show it to anyone: what must be fixed first (security, dependencies with known holes, secrets), what can wait, and a plain ready or not yet, checked in the code itself. Uses the cca MCP server's tools."
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

The task: say whether this repository is ready to ship, and what must happen first if it is not.
1. audit: the scan, the health, the plan and the queue (if resolutionsRefused is not zero, earlier fixes did not hold: those come first).
2. list_findings for the security findings, then the dependencies with known vulnerabilities and any secret in the code; get_finding for each that is not plainly clear.
3. Open every file those findings name and read the code around them: say whether each can be reached by a user or from outside, and what fixing it takes. A finding that does not hold, dispute_finding with your evidence.
4. If there are uncommitted changes, check_dependencies with `git diff -U100000 HEAD`, and read what it says about each new dependency between modules.
5. Read how the application is configured and started (environment files, build and deployment files) for what the scan names, and for debug settings or test credentials that would ship with it.
Write: one line first, "Ready" or "Not yet", and why; then "Fix before shipping" (each item: where, why it matters, what to do, in that order); then "Can wait" (the rest of the plan's top, briefly); then what was not checked. Offer to fix the first list now, one item at a time, with resolve_finding after each and one request_rescan at the end.
