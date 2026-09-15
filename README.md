# Judgement in the Loop

A way of working with an AI coding agent where the agent does the work and I keep the decisions.

I'm a product person with an engineering past. I wrote production code for years, led a team, and then moved to owning what gets built and why. That move taught me something I keep coming back to: in any project, the important decisions get made by whoever is typing at the moment they come up, and everyone else finds out later. Coding agents make that worse. They type faster than anyone, they never pause to ask, and their output looks finished. This is how I stop the decisions from quietly moving to the agent.

![Judgement in the Loop](docs/judgement-in-the-loop.png)

Interactive version: [docs/judgement-in-the-loop.html](docs/judgement-in-the-loop.html), open it in a browser. Made with [archify](https://github.com/tt-a1i/archify); the source is in [docs/judgement-in-the-loop.json](docs/judgement-in-the-loop.json).

## Where the phases come from

Dex Horthy's Research, Plan, Implement split agent work into three prompts so you could read the plan before the code existed. Matan Shavit's [QRSPI](https://github.com/matanshavit/qrspi) added two phases in front of and inside that: Questions first, because the agent should surface the open decisions before it reads anything, and Structure between Research and Plan, because by the time a long plan lands the agent has already made every design decision inside it.

I use QRSPI's five phases as they are. Everything below is what I built around them, and why.

## The day I noticed

Last summer I let an agent design and implement a database schema in one go. It came out fine. A few weeks later I couldn't edit a list item in the app it had built, and when I went looking for the reason I realised I'd never looked at the data model before it existed. The code was good. What had gone wrong was me: I'd stopped noticing where the decisions were being made.

A phase list on its own doesn't fix that. You can run all five phases and still nod through Structure because the draft looks right and you're busy. So everything I added has one purpose: the decisions that are mine pass through me, and the ones that aren't don't eat my attention.

## What I changed

### 1. The agent doesn't get to decide what counts as a decision

In Questions, the agent drafts and I answer. My test for a real question is simple: if it can be answered by reading more code or documentation, it's research wearing the wrong label, and it goes back to the agent. What's left is genuinely mine, and it's always a short list. The stack. The scope. What the feature is for. Which way the design leans.

Why: an agent will happily ask me twenty questions, most of which it could answer itself, and I'll answer them because they're in front of me. The filter keeps my attention on the few that matter, which is the same job I do with a team when I refuse to be the person who answers everything.

### 2. Structure is a stop, and I don't waive it

The agent drafts the structure (the data model, the boundaries between parts, how the thing behaves) and doesn't continue until I've looked at it and said yes. I hold this even when the draft is probably right, and especially when the change feels small.

Why: this is exactly the phase that failed in the schema story. A review I can skip when I'm busy is a review I will skip when I'm busy. A stop with no exceptions costs less than deciding each time whether this one needs it.

### 3. A rules file instead of repeating myself

Before any code I write the non-negotiables into a file the agent reads at the start of every session. On a recent five-day project it had fourteen lines. The API contract is read-only. Deliver what was asked before any extra. The domain imports no framework. Locks are always taken in the same order. Every business rule maps to a test named after it. Errors have one shape. One feature per branch, nothing straight to main.

Why: rules in a file are cheaper than rules in my head, and an agent reads a file more reliably than it remembers a conversation. When it wants to do something outside the rules, it says so and asks, which is the behaviour I'd want from a new colleague too. I write the rules so the build can check as many of them as possible, because a rule nobody checks drifts. And when a review finds a bug, the fix usually becomes a new rule so the bug can't come back.

### 4. Attack before merge

After Implement, separate agents get adversarial prompts with one narrow target each. Find race conditions. Grade the work against the brief. Review it as a tech lead who wants to say no. Edit the docs for plain language. Findings go back to Implement as their own branches, and only green with evidence goes to main.

Why: I can't review my own blind spots, and a general "review this" prompt produces general remarks. A hostile prompt with one target produces specific findings. On that five-day project the races prompt found two real concurrency bugs I'd read past, and the tech-lead prompt produced three push-backs I then had answers for. I wouldn't have found any of it by reading the code again, and I know that because I had.

### 5. The tier is chosen by blast radius

There are two tiers. A small, reversible change (a bug fix, a one-line correction, anything cheap to notice and cheap to undo) runs Research and Implement on its own and reports afterwards. Anything that touches a data model, a contract, an architecture or a real feature goes through the whole loop.

Why: diff size is a poor guide to risk, and it's the guide everyone uses. One line in a migration takes the full loop. Twenty lines in a test don't. The two questions I ask instead are how reversible the change is and how far the damage would reach if it's wrong. Some things override both tiers every time: real money or orders, deleting real data, pushing to anything shared, credentials, anything I can't undo.

### 6. Done means evidence

An agent rounds up. It says "done" when something is verified up to a point, and it diagnoses errors with the same confidence whether it's right or wrong. So every merge carries its evidence: the test run, the fresh boot, the check that the thing works. Once a merge went through with a failing test because a grep had hidden the exit code, so now the exit code is checked explicitly. Once an agent told me a Docker image didn't exist for a platform, and the real cause was a cache on my laptop.

Why: I want evidence before I say "done", and evidence before I say "broken". The agent's report is one input to that.

## A change, start to finish

Say the change is a new endpoint with a state rule behind it. The agent drafts questions. I answer the two that are mine (what the endpoint is for, and whether the rule applies to existing records) and send the rest back as research. The agent reads the contract, the existing model and the tests, and reports what it found. It drafts the structure: where the rule lives, what changes in the data model, what the error looks like when the rule is broken. I look at it, say yes, and one of my decisions goes into the decisions log with its reason. The agent writes a plan and I approve it. It implements on a branch, inside the rules file, with the tests green. Two adversarial agents attack the branch, one for races and one for contract drift. One finding comes back, gets fixed on the same branch, and the branch merges with the evidence attached.

The same shape turns up in the products I build with models inside them: the model proposes, code decides, and a validator sits between the two.

## How this could work in a team

I haven't run this with a team yet. Here's how I'd try, and where I expect it to bend.

The rules file becomes the team's, and it gets reviewed like code. Anyone can propose a rule; the tech lead merges it. The interesting part is that the file makes the team's unwritten agreements written, and the first version will start arguments about things everyone thought were settled. Those arguments are cheaper in a file review than in an incident.

Questions get routed to the person who owns the answer. Product questions go to the product owner, structural ones to the tech lead, and the agent learns which is which from the rules file. The Structure stop belongs to whoever is accountable for the system's shape, which in most teams is the tech lead, and the product owner's stop is one phase earlier, at Questions.

Attack agents run in the pipeline as required checks, with the prompts kept in the repo next to the tests. A finding blocks the merge the way a failing test does. The decisions log becomes the team's memory, so a new person can read why the system looks the way it does without asking.

The tier rule already exists in most teams under another name: the small PR versus the design document. Blast radius just gives it a criterion people can apply without a meeting.

## And in an enterprise

This is further from anything I've tested, so treat it as a sketch.

Rules files could inherit. A platform team owns a baseline (security, error shapes, logging, what an agent may never touch) and each repository adds its own lines. Attack prompts become a shared library: contract drift, data privacy, accessibility, the security review that today happens in a ticket queue. A merge that carries its evidence answers most of what an audit asks for, because the evidence was produced at the time and attached, rather than reconstructed later.

The blast-radius rule is the governance conversation worth having. Most large organisations say "every change needs two reviewers" because they can't tell which changes are dangerous. A shared definition of blast radius would let them spend human review where it matters and let the small tier through.

The risk is obvious: all of this can turn into ceremony within a quarter. The small tier is the safeguard, and it only works if someone defends it.

## What I'm still exploring

- Whose Structure sign-off counts when three humans share one agent and one rules file, and what happens when the product owner and the tech lead disagree at that stop?
- Does Attack pay for itself on small changes, or only on features? So far it's cost about an hour per round and found bugs each time. I don't know where the floor is.
- Should the Structure stop loosen as trust builds with one codebase, and what evidence would justify that?
- How much of this is specific to Claude Code, and how much survives a different agent?
- Where does the product owner sit in the loop when they can't read the code? My answer today is Questions and the decisions log. I'm not sure it's enough.

## Credits and licence

QRSPI is by [Matan Shavit](https://github.com/matanshavit/qrspi); Research, Plan, Implement is Dex Horthy's. The diagram was made with [archify](https://github.com/tt-a1i/archify) by tt-a1i. An example rules file is in [examples/rules-file.md](examples/rules-file.md). Text and diagram in this repository are by Sofia Traba and released under [CC BY 4.0](LICENSE).
