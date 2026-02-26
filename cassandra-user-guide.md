# Cassandra User Guide

## What is Cassandra?

Cassandra is a decision tool that helps you figure out whether a large language model (LLM), like ChatGPT, Claude, Copilot, or Gemini, is the right tool for a particular task, and if so, what checks you need in place to trust the results.

It doesn't produce a risk score or a gradient. It gives you one of three answers:

- **Go ahead** with specific checks in place
- **Go ahead with caution** with conditions you need to meet
- **Don't use an LLM for this** and here's why

Some tasks simply aren't a good fit for LLMs. Cassandra will tell you that directly.

---

## Two versions

Cassandra comes in two forms:

**Cassandra** (web version) is a self-contained decision tree that works in any browser. No internet connection, login, or API access required. You answer each question manually. Best for hosting on an intranet, sharing as a link, or using where LLM access isn't available.

**Cassandra for Agents** assesses multi-step AI agent workflows rather than single tasks. You describe a workflow, an LLM decomposes it into steps, and the tool walks you through a risk assessment of the full chain: which steps need an LLM, how each step is validated, what happens when things go wrong, and where human checkpoints belong. It runs as a Claude artifact.

The two tools share the same underlying logic for assessing individual LLM tasks. The Agents version adds workflow-level questions (consequence, error propagation, checkpoints, logging) that only apply when steps are chained together and the agent takes actions.

---

## How it works

You describe a task you're considering using an LLM for. Cassandra then walks you through a series of questions, between 1 and 7 depending on the path:

1. **Does accuracy matter?** If not, you're clear to use an LLM freely.
2. **Is the LLM producing the output itself, or using tools?** An LLM generating a report is different from an LLM that writes a database query and summarises the results. Each has its own path.
3. **What kind of task is this?** Labelling, code, summarising/extracting, analysis, or other. Each has different verification questions.
4. **Can you verify the output?** The specific questions depend on the task type.

The questions branch depending on your answers, so you only see what's relevant.

---

## Writing a good task description

The quality of Cassandra's assessment depends on the quality of your task description.

### What to include

A good task description answers these questions:

**What will the LLM produce?**

Be specific about the output. "A report" is vague. "Weekly sales reports showing conversion rates, revenue by territory, and pipeline metrics" is specific.

**Where does the data come from?**

Mention whether the LLM will be working from data you provide (a spreadsheet, database, document set) or generating from its own knowledge. This is the single most important distinction.

**Who will use the output, and how?**

Will this inform decisions? Be presented to leadership? Feed into a dashboard? Or is it a rough draft for your own thinking?

**Is this a one-off or recurring task?**

A one-off analysis has different risks than an automated weekly pipeline. Say which it is.

**What checks are already in place?**

If you have test suites, validation scripts, training data, or review processes, mention them. These directly affect the outcome.

### Examples of good descriptions

> "Sort 44,000 employee survey comments into predefined categories (positive, negative, mixed, neutral). We have 500 hand-labelled examples from last year's survey and can lock the model version."

This tells Cassandra: it's a labelling task, there's training data, and the model can be frozen. That's enough to assess it.

> "An LLM writes a data processing script. We have a full test suite with known inputs and expected outputs, plus automated security scanning before anything goes live."

This tells Cassandra: it's code, it's testable, and there's security review. Clear path.

> "Summarise 200 customer support tickets from last week into a trends report highlighting the most common issues. A person reads the summary and decides if it reflects what they saw in the queue."

This tells Cassandra: it's summarisation from source material, with human checking, likely recurring. It will flag that the human checking will fade over time.

> "We want an LLM to explain why customer churn went up 12% last quarter and suggest what to do about it, using CRM data, support tickets, and market research."

This tells Cassandra: it's analysis, the output is narrative and interpretive, and the "using CRM data" part doesn't change the fact that the LLM is being asked to interpret, not just count.

### Examples of poor descriptions

> "Use an LLM for sales"

Too vague. What specifically? Forecasting? Report generation? Lead scoring? Email drafting?

> "Analyse our data"

What data? What kind of analysis? Who uses the results? What decisions depend on it?

> "Generate reports"

What kind of reports? From what data? How often? Who reads them?

### The key details that change the outcome

These specific details can shift Cassandra's verdict from "don't use" to "go ahead":

| Detail | Why it matters |
|--------|---------------|
| "We have 500 labelled examples" | Opens the training data path for labelling tasks |
| "We can pin the model version" | Prevents silent accuracy degradation |
| "We can spot-check a sample against human labels" | Gives a measure of accuracy even without training data |
| "We have automated tests" | Makes code generation defensible |
| "We run automated security scanning" | Closes the security gap for code |
| "A script compares output to source data" | Provides faithfulness checking for summarisation |
| "We can check against real data" | Provides structural verification for analysis |
| "This is a one-off task" | Human verification is acceptable for one-offs |
| "This runs weekly" | Human verification will erode: needs automation |

---

## Understanding the outcomes

### Go ahead

The task has appropriate structural checks. The LLM's output is verified through independent mechanisms, not by someone reading it and deciding it "looks right."

### Go ahead with caution

The task is viable but has a specific gap. The outcome will explain what the gap is (usually a missing security review, reliance on human verification for a one-off task, or a checking process that will degrade with repetition).

### Don't use an LLM for this

The task can't be adequately verified, or the verification depends on human attention that will erode over time. The outcome explains why and suggests what to do instead.

---

## The decision tree at a glance

```
Does accuracy matter?
+-- No -> GO AHEAD (freely)
+-- Yes -> How is the LLM used?
    +-- Orchestrating (calling tools) -> Can you see what it asked?
    |   +-- No -> DON'T USE
    |   +-- Yes -> Can you check the tool's raw output?
    |       +-- No -> CAUTION (trusting the interpretation)
    |       +-- Yes -> Is the chain logged?
    |           +-- No -> DON'T USE
    |           +-- Yes -> Automated or human review?
    |               +-- Automated -> GO AHEAD
    |               +-- Human -> Repeating? ...
    +-- Producing output -> What kind of task?
        +-- Labelling -> Have training data?
        |   +-- No -> Can you spot-check a sample?
        |   |   +-- No -> DON'T USE
        |   |   +-- Yes -> CAUTION (measure before you trust)
        |   +-- Yes -> Can freeze model version?
        |       +-- No -> DON'T USE
        |       +-- Yes -> GO AHEAD (with accuracy checks)
        +-- Code -> Can write automated tests?
        |   +-- No -> DON'T USE
        |   +-- Yes -> Security review?
        |       +-- No -> CAUTION (security gap)
        |       +-- Automated -> GO AHEAD
        |       +-- Human -> Repeating?
        |           +-- One-off -> GO AHEAD (with review)
        |           +-- Recurring -> CAUTION (review will erode)
        +-- Summarising/extracting -> Can check against source?
        |   +-- No -> DON'T USE
        |   +-- Yes -> Automated or human checks?
        |       +-- Automated -> GO AHEAD
        |       +-- Human -> Repeating?
        |           +-- One-off -> CAUTION (check this one)
        |           +-- Recurring -> CAUTION (checking will fade)
        +-- Analysis -> Can verify independently?
        |   +-- No -> DON'T USE
        |   +-- Yes -> Automated or human checks?
        |       +-- Automated -> GO AHEAD
        |       +-- Human -> Repeating?
        |           +-- One-off -> CAUTION (verify this one)
        |           +-- Recurring -> DON'T USE (checking will fade)
        +-- Other -> CAUTION (assess manually)
```

---

## Background

Cassandra was inspired by a story shared on Reddit describing an LLM agent that reportedly fabricated sales analytics for three months. The VP of Sales made territory decisions based on this data. The CFO presented it to the board. An analyst who raised concerns was told they were "slowing down innovation." The data turned out to be completely made up.

Whether or not the specific story is accurate, the pattern it describes is real and well-documented: LLM-generated output that looks professional and plausible, presented without structural verification, in a pipeline where human oversight erodes over time.

Cassandra exists to help people recognise when they're in that pattern, before the damage is done.
