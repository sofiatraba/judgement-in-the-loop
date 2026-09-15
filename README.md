# Judgement in the Loop

How I build software with an AI coding agent without handing over the decisions.

The base is [QRSPI](https://github.com/matanshavit/qrspi) by Matan Shavit: Questions, Research, Structure, Plan, Implement. QRSPI grew out of Dex Horthy's Research, Plan, Implement. I did not invent the phases. What I added is the part that keeps my judgement in the loop while the agent does the work: a rules file, a checkpoint that cannot be skipped, adversarial agents before every merge, and a rule for when to use the full loop at all.

![Judgement in the Loop](docs/judgement-in-the-loop.png)

Interactive version: [docs/judgement-in-the-loop.html](docs/judgement-in-the-loop.html) (open the file in a browser; made with [archify](https://github.com/tt-a1i/archify), source in [docs/judgement-in-the-loop.json](docs/judgement-in-the-loop.json)).

## The moment it started

July 2026. I was building a meal-planning app that talks to a real grocery basket, as a personal tool and as a lab for how an AI product manager should build. I asked the agent to design and implement the database schema. It did both in one go. The schema came out fine.

Then I asked for an MCP layer so the agent could act on the app more autonomously, and I realised I could not edit a list item in the app I had already had it build. I had been letting Implementation run ahead of Structure, again and again, without noticing. "Fine" had been luck. I had never looked at the data model before it existed.

That is the failure mode I care about. The agent did not write bad code. I stopped noticing where the decisions were being made.

## What I believe

AI should remove friction, and it should never replace judgement. Reading library source, migration guides and existing code before touching anything is friction; I do not need to be in the loop for that. Deciding the shape of a data model, the trade-off in an algorithm, what a feature should actually do: that is judgement, and judgement is the part of my job that does not transfer to the agent no matter how good its output looks.

## The loop

| Phase | Who owns it | Why |
|---|---|---|
| **Questions** | Agent drafts, I answer | If a question can be resolved by reading more code, it is research with the wrong label. Real questions are mine to decide: the stack, the scope, the design direction. |
| **Research** | Agent, autonomous | Breadth reading is where an agent is faster than my oversight would add value. Facts only, no opinions about implementation. |
| **Structure** | Agent drafts, I validate before it continues | The gate that was missing in July. Architecture and behaviour decisions are mine to own, on purpose, even when the draft is probably right. |
| **Plan** | Agent writes, I approve or redirect | A written, reviewable artefact. Cheaper to correct a paragraph than a component tree. |
| **Implement** | Agent, autonomous, inside the rules file | Tests, build and a real check that it works live inside this step. |
| **Attack** | Adversarial agents with a target | Find races. Grade against the brief. Edit for plain language. Findings go back to Implement; green plus evidence goes to main. |

Two tiers. A small, reversible change (a bug fix, a one-line correction, anything cheap to notice and cheap to undo) runs Research then Implement autonomously and reports after. Anything touching a data model, a contract, an architecture or a real feature goes through the whole loop. The criterion is reversibility and blast radius, never the size of the diff. One line in a migration takes the full loop; twenty lines in a test do not.

What overrides both tiers, every time: real money or orders, deleting real data, pushing to anything shared, credentials, anything I cannot undo.

## What a take-home test added

September 2026, a take-home for a Product Engineer role at a large retailer: a supplier lifecycle API from an OpenAPI contract, a dashboard, one-command boot, five days. It was the first project where I ran the loop end to end under time pressure, and three things became rules.

**The rules file.** Before any code I wrote fourteen non-negotiables in a `CLAUDE.md` that the agent reads at the start of every session: the contract is read-only, deliver the brief before any extra, Docker only, the domain imports no framework, locks are always taken in the same order, every business rule maps to a test named after it, errors always have one shape, one feature per branch, nothing straight to main. I never had to repeat them. When the agent proposed something outside them, it said so and asked. The file shipped with the delivery so the reviewers could see the contract I had with the agent. A lightly anonymised copy is in [examples/rules-file.md](examples/rules-file.md).

**Attack before merge.** After the required scope was done I gave separate agents adversarial prompts: find race conditions, grade this against the brief using the recruiter's own tips, review it as a tech lead who wants to say no, edit the docs for plain language. The races prompt found two real bugs I had read past. Applying and refusing on the same candidate at the same time could leave a refused candidacy next to a live supplier. Accepting and banning in the same country could deadlock. Both became locks taken in a fixed order, with tests that race the requests. The tech-lead prompt produced three push-backs I then prepared answers for. None of those findings would have come from me reading my own code again.

**Verified means verified.** The agent rounds up. It says "done" when a thing is verified up to a point. Once a merge went through with a failing test because a grep had masked the exit code; now the exit code is checked explicitly before every merge. After submitting, an emulated Intel boot failed with an error that looked like a missing platform in the image; the real cause was a cached image on my laptop. The lesson goes both ways: evidence before "done", and evidence before "broken".

I also rejected things. The agent could have faked "filter the whole ranking" by fetching many pages behind the user's back; the contract capped pages at ten, so the UI says "this page" and the fix went into the backlog as a contract change. The first extra page had explanatory copy that read like a model wrote it; I cut it. Facts about the test mock were about to appear in product UI; they went to the docs.

## The same idea in smaller projects

**A training-load tool** over my Strava history. The script decides every threshold, zone and recommendation. The model only phrases the weekly report from the evidence the script prints. If a report is ever wrong, the fix is in the script or in the prompt file, never in a one-off correction.

**The meal-planning app.** The model picks the week's meals; a validator rejects any recipe id it invents, and a test feeds it a made-up id to prove it. "Propose" returns a draft and writes nothing; "apply" is a separate call. I built the version without that step first, because it demos better, and only noticed when I tried to trust it.

Same rule each time: the model proposes, code decides.

## What I refuse to do

- Let an agent make an architecture or data-model call unreviewed because the output looked plausible. Plausible is the wrong bar.
- Skip the Structure checkpoint because the change seems small. Size is the wrong criterion.
- Treat "the agent already built it" as evidence it is right. Sunk cost is a feeling.
- Turn this into ceremony. The small tier exists so a typo fix does not go through six phases.
- Merge on "done" without the evidence attached.

## Still testing

- Does the loop hold in a team? Whose Structure sign-off counts when three humans share one agent and one rules file?
- Does Attack pay for itself on small changes, or only on features? On the take-home it cost about an hour per round and found bugs each time. I do not know the floor.
- Does the Structure gate earn the right to loosen as trust builds with one codebase, and on what evidence?
- How much of this is specific to Claude Code and how much survives a different agent?

## Credits and licence

QRSPI is by [Matan Shavit](https://github.com/matanshavit/qrspi). The diagram was made with [archify](https://github.com/tt-a1i/archify) by tt-a1i. Text and diagram in this repository are by Sofia Traba and released under [CC BY 4.0](LICENSE).
