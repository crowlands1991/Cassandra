# Cassandra
Most AI failures in the workplace aren't caused by bad models. They're caused by missing checks. Someone asks an LLM to do something, the output looks convincing, and nobody verifies it. Cassandra helps you catch that gap before it causes a problem.
What it does
Cassandra is a decision tree that assesses whether a specific LLM use case has the right safeguards in place. You describe your task, answer questions about how the output will be checked, and get one of three verdicts: go ahead, proceed with caution, or don't use an LLM for this.
It covers five task types: sorting/labelling data, summarising/extracting, writing code, analysis and written output, and LLM-as-orchestrator (where the LLM calls tools on your behalf). Each path asks different questions because the risks are different.
What it checks for
The core question behind every path is the same: if the LLM gets this wrong, how would you know?
Cassandra looks at whether you have independent verification (not just a person reading the output and deciding it "looks right"), whether that verification is automated or relies on human attention, and whether the task is one-off or recurring. Recurring tasks with human-only checking get flagged, because people check carefully at first, then skim, then stop.
How to use it
Open the page. Describe your task. Answer the questions. Each outcome includes specific guidance and a challenge question to pressure-test your setup.
No login, no data collection, no dependencies. It runs entirely in your browser.
Background
Cassandra was built to give teams a practical, non-technical way to assess LLM risk at the point of use, not as a policy document that sits in a drawer. The name comes from the Greek myth: she could see the future but nobody listened. The tool's job is to make the warnings hard to ignore.
v3.1 | 25 Feb 2026
