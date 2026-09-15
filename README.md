# Judgement in the Loop

A way of working with an AI coding agent where the agent does the work and I keep the decisions.

![Judgement in the Loop](docs/judgement-in-the-loop.png)

Interactive version: [docs/judgement-in-the-loop.html](docs/judgement-in-the-loop.html), open it in a browser. Made with [archify](https://github.com/tt-a1i/archify); the source is in [docs/judgement-in-the-loop.json](docs/judgement-in-the-loop.json).

## Where the phases come from

Dex Horthy's Research, Plan, Implement split agent work into three prompts so you could read the plan before the code existed. Matan Shavit's [QRSPI](https://github.com/matanshavit/qrspi) added two phases: Questions in front, because the agent should surface the open decisions before it reads anything, and Structure between Research and Plan, because by the time a long plan lands the agent has already made every design decision inside it.

I use QRSPI's five phases as they are. What follows is what I built around them, and why.

## Why I needed more than the phases

I let an agent design and implement a database schema in one go. It came out fine. Weeks later I couldn't edit a list item in the app it had built, and I realised I'd never looked at the data model before it existed. The code was good. The problem was that I'd stopped noticing where the decisions were being made, and a phase list on its own doesn't stop that. You can run all five phases and still nod through Structure because the draft looks right.

So the additions are all about one thing: making sure the decisions that are mine actually pass through me, and that the ones that aren't don't waste my attention.

## What I changed

### 1. The agent doesn't decide what counts as a decision

In Questions, the agent drafts and I answer. My test for a real question: if it can be answered by reading more code or documentation, it's research wearing the wrong label and goes back to the agent. What's left is genuinely mine, and it's always a small list: the stack, the scope, the design direction, what the feature is for.

Reasoning: an agent will happily ask me twenty questions, most of which it could answer itself, and I'll answer them because they're there. The filter keeps my attention on the few that matter.

### 2. Structure is a stop, and I don't waive it

The agent drafts the structure (the data model, the boundaries, the behaviour) and doesn't continue until I've validated it. I hold this even when the draft is probably right, and especially when the change seems small.

Reasoning: this is the phase that failed in the schema story. A review I can skip when I'm busy is a review I will skip when I'm busy. Making it a stop with no exceptions is cheaper than deciding each time whether this one needs it.

### 3. A rules file instead of repeating myself

Before any code I write the non-negotiables into a file the agent reads at the start of every session. On a recent five-day project it had fourteen lines: the API contract is read-only, deliver the brief before any extra, the domain imports no framework, locks are always taken in the same order, every business rule maps to a test named after it, errors have one shape, one feature per branch, nothing straight to main.

Reasoning: rules in a file are cheaper than rules in my head, and the agent reads a file more reliably than it remembers a conversation. When it wants to do something outside the rules it says so and asks, which is the behaviour I want. I write the rules so the build can check as many of them as possible, because a rule nobody checks drifts. When a review finds a bug, the fix often becomes a new rule so it can't come back.

### 4. Attack before merge

After Implement, separate agents get adversarial prompts with a narrow target: find race conditions, grade the work against the brief, review it as a tech lead who wants to say no, edit the docs for plain language. Findings go back to Implement as their own branches. Only green with evidence goes to main.

Reasoning: I can't review my own blind spots, and a general "review this" prompt produces general remarks. A hostile prompt with one target produces specific findings. On that same five-day project the races prompt found two real concurrency bugs I'd read past, and the tech-lead prompt produced three push-backs worth preparing answers for. None of that would have come from me reading the code again.

### 5. The tier is chosen by blast radius

Two tiers. A small, reversible change (a bug fix, a one-line correction, anything cheap to notice and cheap to undo) runs Research and Implement on its own and reports afterwards. Anything that touches a data model, a contract, an architecture or a real feature goes through the whole loop.

Reasoning: diff size is a poor guide to risk. One line in a migration takes the full loop; twenty lines in a test don't. What I ask is how reversible the change is and how far the damage would reach. Some things override both tiers every time: real money or orders, deleting real data, pushing to anything shared, credentials, anything I can't undo.

### 6. Done means evidence

An agent rounds up. It says "done" when something is verified up to a point, and it diagnoses errors with the same confidence whether it's right or wrong. So a merge carries its evidence: the test run, the fresh boot, the check that the thing works. Once a merge went through with a failing test because a grep had hidden the exit code; now the exit code is checked explicitly. Once an agent told me an image didn't exist for a platform, and the real cause was a cache on my laptop.

Reasoning: I want evidence before I say "done", and evidence before I say "broken". The agent's report is one input to that.

## A change, start to finish

Say the change is a new endpoint with a state rule behind it. The agent drafts questions; I answer the two that are mine (what the endpoint is for, whether the rule applies to existing records) and send the rest back as research. The agent reads the contract, the existing model and the tests, and reports facts. It drafts the structure: where the rule lives, what changes in the data model, what the error looks like. I validate it, and one of my decisions goes into the decisions log. The agent writes a plan; I approve it. It implements on a branch, inside the rules file, with the tests green. Two adversarial agents attack the branch: one for races, one for contract drift. One finding comes back, gets fixed on the same branch, and the branch merges with the evidence attached.

The same shape shows up in the products I build with models inside them: the model proposes, code decides, and a validator sits between them.

## What it's for

Solo work and small teams where an agent does most of the typing. It's built to keep judgement where it belongs while giving the agent everything else. It isn't ceremony: the small tier exists so a typo fix doesn't go through six phases.

## Open questions

- Whose Structure sign-off counts when three humans share one agent and one rules file?
- Does Attack pay for itself on small changes, or only on features? So far it's cost about an hour per round and found bugs each time. I don't know where the floor is.
- Should the Structure stop loosen as trust builds with one codebase, and what evidence would justify that?
- How much of this is specific to Claude Code, and how much survives a different agent?

## Credits and licence

QRSPI is by [Matan Shavit](https://github.com/matanshavit/qrspi); Research, Plan, Implement is Dex Horthy's. The diagram was made with [archify](https://github.com/tt-a1i/archify) by tt-a1i. An example rules file is in [examples/rules-file.md](examples/rules-file.md). Text and diagram in this repository are by Sofia Traba and released under [CC BY 4.0](LICENSE).
