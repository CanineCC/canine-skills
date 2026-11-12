---
name: due-diligence
description: "A due diligence memo on one codebase: what you would inherit, the risks that carry a price, what to ask the seller, and what the seller should fix or disclose before closing, from its decision report and, where you have the code, your own reading of it. Uses the assay MCP server's tools."
---

<!-- Generated from the same source as the assay server's MCP prompt of the same name; do not edit by hand. -->

The task: a due diligence memo on the codebase the person names (list_codebases; ask which, if there are several) for the people deciding on the deal.
1. list_codebases, then get_codebase: its score from signed evidence and from the latest scan, who supplied the evidence, and its reports.
2. get_report for the newest decision report: the verdict, whether the evidence signature verified and the verdict reproduced, the trend, and the report's text. Build on its text and never contradict it.
3. explain_dimension for every dimension and lens the report names as weak: what it affects in a deal and who should care.
4. If the person has the code open, the report is your map: open what its weakest dimensions point to (the modules, the dependency files, the configuration, the tests) and read it, to say what fixing each would take. Keep what the report measured apart from what you saw yourself, and name the files you read. If you do not have the code, say so and do not guess what is in it.
Write, in plain words for people who do not read code: the verdict in three sentences, with the report's date and whether its evidence verified; what they would inherit (what holds, what does not); the risks that carry a price (each with what it affects: price, delivery, security, people) ranked by cost; the questions to ask the seller, each tied to a finding; what the seller should fix or disclose before closing; what was not measured. Do not explain how the score is counted; say what was found and what it means for the deal.
