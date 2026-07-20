# Chapter 14: Self-Evolving Agents

*Draft in the author's voice. Part III, final pattern chapter. Per author: this chapter combines the loop with memory, the way the dream agents do. Deflates "self-evolving" early (the weights never move; it is a background job that writes notes) and makes that the safety story rather than a disappointment. Patterns held to 2-3 sentences. Running-example is a TODO block. In the wild leads with Google. No em-dashes; a few ellipses. Which phase: the loop, closed through memory.*

---

## What happens between sessions

The loop ends. The context window is discarded. Everything the agent worked out during those forty turns, the dead end it wasted twenty minutes in, the fix that finally worked, the convention it inferred from your codebase, goes with it.

Tomorrow it starts again from the same blank slate, and makes the same mistake at the same place. Your agent is not learning. It is auditioning, every single day, for a job it has already done a thousand times.

That gap between one loop and the next is what this chapter fills. You have a loop that runs and a memory that stores, and nothing yet connects them into a circuit where running produces experience, experience becomes knowledge, and knowledge changes the next run.

## Consolidate, distil, generalise

The loop runs and produces **episodes**: what was attempted, what came back, what worked. They pile up, and they are mostly noise.

**Background Reflection.** *Read many episodes during idle time and write down the few things that generalize.* Point it at the sessions since it last ran, have it look for repeated patterns, corrections, and failures, and let it produce a small number of durable notes. Nothing about the current run improves, which is the point: this is the work you do when nobody is waiting.

The timing is not incidental. Consolidation is slow, costs real tokens, and helps nothing in flight, so it belongs off the hot path in the gap when nobody is asking the agent for anything. Everyone who has built this reached for the same metaphor and named it after sleep.

Be precise about what comes out, because the field is sloppy here. Storing every transcript and retrieving the relevant one is retrieval, and it is a good thing to have. Evolution is the narrower thing: converting experience into a persistent change in future behaviour. Plenty of systems retain. Far fewer convert. What consolidation hands you is a lesson, and the rest of this chapter is about where to put it.

## Six things you can change

Consolidation gives you a lesson. The question nobody answers is where to put it.

An agent has six places a lesson can land, and they form a ladder. At the bottom, a note in a memory file. At the top, the model's weights. Every rung up buys more permanence and costs you more of everything else.

As you climb, the **blast radius** grows: a note affects the runs that retrieve it, a system prompt affects every run ever. **Reversibility** falls: deleting a line takes a second, and un-learning a fine-tune takes a retrain. **Feedback slows**: a note works tonight, a tuned model works in three weeks. **Auditability** drops: you can read a runbook, and you cannot read a weight. And the **cost** rises from nothing to real money.

So the discipline is simple to state and widely ignored: put the lesson on the lowest rung that can hold it. Most teams reach for the top rung first, because it is the one that sounds like machine learning.

## The rungs

**Updated Notes.** *Write the fact into memory and let retrieval carry it.* The cheapest rung and the right default: this service is owned by that team, this customer prefers email. It applies the moment it is written, it costs nothing, and a human can delete it in one keystroke when it turns out to be wrong.

**New Skills.** *Promote a procedure that worked into a named, retrievable skill.* Where a note is a fact, a skill is a routine: the checklist that resolved the incident, the sequence that passed the tests. One distilled skill stands in for the twenty raw trajectories it came from, which is what makes the library affordable to carry.

**Rewritten Instructions.** *Fold the lesson into the prompt that every run starts with.* Use this for the things that are always true and should never need retrieving: the conventions, the house style, the standing constraints. Watch the blast radius, because unlike a note this reaches every run whether it is relevant or not, and the prompt is a shared resource that fills up.

**New Tools.** *When the agent keeps doing the same thing by hand, have it write a tool for it.* A procedure the agent re-derives every time is a tool waiting to happen, and turning it into one moves it from red to black: now it is code, so it is testable, deterministic, and free. The catch is that this is code your agent wrote, so it belongs behind a sandbox and a review.

**Changed Code.** *Push the lesson into the deterministic skeleton.* The router learns a new category, a threshold moves, a workflow grows a branch. This is the highest-leverage rung, because it changes the parts of the system that were reliable to begin with, and for exactly that reason it should almost always arrive as a pull request a human merges rather than something the agent does at 3 a.m.

**A Tuned Model.** *Distil the trajectories into training data and move the weights.* Collect the runs, filter them hard, and fine-tune on what survives, which is the standard recipe and the only rung that changes the model itself. Reach for it when a behavior needs to be instinct rather than instruction, and understand that you are trading every property in the list above, auditability most of all, for a capability the other five rungs cannot reach.

## The filter is the whole job

The top rung deserves its own warning, because it is the one where the details decide whether you improve or poison the model.

Training on your own trajectories means training on your own mistakes unless you filter, and a failed run teaches the model to fail. The standard answer is to keep only the trajectories that succeeded, which works and is brutally wasteful: one well-known agent dataset threw away roughly sixty percent of its collected runs to get a clean training set.

Look at what a failed run actually contains. The agent read the right files, ran the right tests, made four sensible edits, then made one bad assumption and burned its remaining turns chasing it. Twenty turns, of which maybe five were wrong. The run failed, so whole-run filtering throws away all twenty, the four good edits included.

Someone measured this properly. In unsuccessful trajectories, only about a quarter of the individual turns were actually mistakes; the other three quarters were decent work attached to a bad ending.

So filter turn by turn instead of run by run. Send the failed trajectory to a critic, have it mark which turns were the mistakes, drop only those, and train on what is left. You keep the good work and throw away the bad moves, which is what you wanted in the first place.

The rule underneath is the one the synthetic-data people repeat until it is boring: an unfiltered dataset is worse than a smaller filtered one. Whatever rung you are on, the quality of what you learn from beats the volume of it.

## The gate

One control applies at every rung, and it is the difference between an agent that improves and an agent that drifts.

**No learned change reaches production without passing evaluation.** Treat a new note exactly like a code change: run it against the golden set from Chapter 9, and if scores regress, it does not ship. This is boring and it is the whole safety story, because every failure below is a change that shipped without anyone checking.

Two rules travel with it. **Keep the learning readable**, in plain language a reviewer can look at and reject, which is why the bottom rungs are safer than the top one: you can argue with a runbook. And **bound what the agent may change about itself**: notes and skills, usually yes; its own guardrails, its policy layer, its termination conditions, never. The distinction is between an agent that gets better at its job and an agent that renegotiates the job.

## What goes wrong

Four failures, and they share a shape: the agent learns something, and the something is wrong.

**A bad lesson persists.** The agent draws a confident conclusion from one unlucky run, writes it down, and now every future loop inherits it. Retrieval spreads the error rather than correcting it, and nothing in the system is looking for a note that is simply false.

**The library rots.** Skills accumulate, the world moves, and the runbook that worked in March is quietly wrong in September. Consolidation adds, and almost nobody builds the thing that removes.

**Drift.** Small changes compound in a direction nobody chose. Each one looked reasonable, and forty of them later the agent behaves differently from the thing you evaluated, which is why the gate matters more than the mechanism.

**Reward hacking.** Ask the agent to improve against a measure and it will improve against the measure, including by learning that shorter answers score better, or that skipping the verification step makes runs faster. Whatever you count, it will optimize, and it will not tell you it found a shortcut.

## TODO: running example

> **TODO (author):** the running-example section goes here, to be written once the running example is settled.
>
> *What this section should do:* Walk the running example up the ladder, since that is the chapter's arc. Background Reflection reads the last two weeks of incidents overnight and notices the same failure four times with the same fix. Now place the lesson: an **Updated Note** for the ownership fact it got wrong, a **New Skill** for the runbook that resolved it, a **New Tool** once it notices it has hand-rolled the same three queries fifteen times, and a **Changed Code** proposal for the router that keeps mis-categorising this alert, which arrives as a pull request a human merges rather than something the agent does at 3 a.m. Say plainly why it never reaches the top rung: the volume is too low, the feedback too slow, and an on-call engineer needs to be able to read why the agent did what it did. Then show the gate firing on every rung, and the bounded-modification line: it may write runbooks and may never touch its own guardrails or the human gate on production writes. Close on the point of the whole thing: the fourth incident should not cost what the first one did, and the thing that makes that true is not a smarter model.

## In the wild

Google Research put the bottom of the ladder in a title: language models need sleep. The work treats consolidation as a first-class part of the architecture rather than a bolted-on feature, learning to compress experience into durable memory between bouts of work, which is the same instinct that shows up in the Antigravity stack as reflective memory and in Vertex as a managed memory service. The framing worth stealing is that idle time is a resource. Every agent has hours a day when nobody is asking it anything, and that is compute you have already paid for.

Anthropic shipped the productized version of the same two rungs in May 2026 as Dreaming: during idle time a managed agent reviews its own past sessions, extracts patterns, and writes memory notes that make the next session start smarter. What makes it interesting is what it refuses to do. No weights move, no gradients flow, and every learned item lands as a human-readable entry a team can review, approve, reject, or edit before it goes live. The trade is stated honestly rather than hidden: the ceiling is whatever memory-augmented prompting can reach, which is lower than retraining could unlock. They took the lower ceiling to keep the audit trail, and an early production user was a legal-drafting firm, where that trade needs no explanation.

The top rung is where the coding agents live, because they produce the cleanest training signal in software: the tests pass or they do not. The standard recipe is to generate many trajectories, keep the ones that succeeded, and fine-tune on those, and its cost is visible in the numbers. One well-known agent dataset discards around sixty percent of its collected runs to get a clean set. Work published in 2026 pushed at that waste and found that in failed trajectories only about a quarter of the steps were actually mistakes, so the rest of that sixty percent was good work discarded for having a bad ending, which is the case for filtering step by step rather than run by run. Meanwhile the same teams keep the bottom rungs running underneath: Claude Code has let the agent write to its own memory file mid-session since late 2025, and the skill-library research through 2026 distils both successes and failures into named, retrievable procedures at roughly ten to twenty times compression against the raw trajectories. Nobody serious is on one rung.

## When not to reach for this

Stability and auditability sometimes beat adaptation, and in a regulated domain they usually do. An agent whose behavior changes between Tuesday and Thursday is an agent whose Tuesday certification means nothing, and "it learned something overnight" is not an answer an auditor accepts. The other case is simpler: if the work is genuinely novel every time, there is nothing to consolidate, and a dream job will spend real money distilling coincidences into rules. Evolution pays off exactly where the work repeats, which is most work, but not all of it.

## Which phase

This is the **loop**, closed. Everything in this part treated a run as a thing with a beginning and an end; here the end feeds the beginning, through memory, and the system stops being a machine that executes and starts being one that accumulates. Notice what did not change: the model, the tools, the guardrails, the gates. The agent improves by writing better notes to itself, under review, which is a modest mechanism for an immodest name and the reason it is safe enough to ship.

---

### Author notes (not for the reader)
- **Running example (standing decision):** running-example section is a TODO placeholder pending the author's decision on the example.
- **Chapter premise (author's brief, revised twice):** this chapter combines **the loop with memory**, the way the dream agents work, plus the recent self-evolving engineering-agent ideas. The author then sharpened the pattern set: after consolidation, distillation, and generalisation, **the patterns should be the ways to evolve an agent**, update a skill, produce synthetic data to train the model, change the deterministic code, and so on. So the catalog is now **the targets of evolution**, not the mechanisms.
- **The six rungs (the catalog), ordered by permanence and risk:** Updated Notes, New Skills, Rewritten Instructions, New Tools, Changed Code, A Tuned Model. As you climb: **blast radius** grows (a note affects retrievals, a system prompt affects every run), **reversibility** falls (delete a line vs retrain), **feedback slows** (tonight vs three weeks), **auditability** drops (read a runbook vs read a weight), **cost** rises. The discipline: **put the lesson on the lowest rung that can hold it.** The deflation is now positional rather than absolute: "Most teams reach for the top rung first, because it is the one that sounds like machine learning."
- **Superseded thesis, do not reinstate:** an earlier draft claimed **"the weights never move"** and made the frozen model the chapter's spine. That was wrong once the author added synthetic-data-for-training as a legitimate rung. The honest frame is the ladder: most evolution never touches the weights, and fine-tuning is the last rung rather than a thing that does not exist. The "background job that writes markdown" line went with it; the auditability argument it carried now lives in the Dreaming paragraph and in The Gate.
- **Retained deflation:** "memory is the substrate, not the definition" survives in *Consolidate, distil, generalise* ("Plenty of systems retain. Far fewer convert."), from the OPD-Evolver framing (arxiv 2606.17628).
- **Naming (author's catch):** the consolidation pattern was called "The Dream Job," rejected as meaningless (a pun on Anthropic's product name that describes nothing). Renamed **Background Reflection**. This is Ch 6's **Reflect** running on Ch 6's background timing axis; the chapter does not pretend it is new, and Google's "reflective memory" is the same thing, so naming is consistent across all three. Watch the Ch 6 / Ch 14 seam if it starts feeling repetitive; Ch 14's contribution is the ladder, not the operation.
- **Sources (dated bucket, verify all before print):** **Google Research, "Language models need sleep: learning to self-modify and consolidate memories"** (Behrouz, Hashemi, Mirrokni, arxiv 2606.03979), the Google-first anchor; pairs with Antigravity reflective memory and Vertex Memory Bank. **Anthropic Dreaming**, ~May 6 2026 at Code with Claude, managed agents; idle-time session review, pattern extraction, memory notes; no weight changes, no gradients; every learned item human-readable and reviewable/approvable/rejectable/editable; hippocampal-consolidation analogy behind the name; ceiling explicitly bounded by memory-augmented prompting; **Harvey** (legal drafting) an early production tester, decide whether to name them. Dario Amodei's "60% probability of models training themselves within two years" deliberately omitted (dates badly, cuts against the chapter). **Top rung:** rejection-sampling fine-tuning (RFT) as the standard agent-training recipe; **SWE-smith discards ~61% of collected runs**; **only ~24% of steps in unresolved trajectories are actually mistakes** (manual analysis of 20 trajectories), motivating step-level filtering with a critic (JetBrains SRFT, arxiv 2605.10674, blog Jun 2026); "an unfiltered synthetic dataset is worse than a smaller filtered one" (futureagi, May 2026); agent fine-tuning now a distinct workflow with tool-use traces scored by a task-adherence judge; closed-loop co-evolution where iteration N's model generates iteration N+1's trajectories (UI-Voyager, arxiv 2603.24533). **Bottom rungs:** Claude Code auto-memory writing to its own file mid-session (~v2.0.64, late 2025); AutoDream (Feb 2026) background consolidation sub-agent; skill libraries SkillRL/SKILLBANK (**10-20x compression**, successes *and* failures, arxiv 2602.08234), SkillWeaver (2504.07079), CODESKILL (2605.25430); SkillsVote admitting only "successful, reusable, and attribution-supported" discoveries. "Failure is useful only when an analytic pipeline turns it into a specific update target" is a good line if a quote is wanted (openreview f78c762).
- **Cross-refs:** memory tiers, Reflect, Prune, the timing axis (Ch 6); the golden set and the eval gate (Ch 9); red-to-black, since New Tools moves a procedure from model to code (Ch 11); the loop (Ch 13); sandboxing for agent-written tools (Ch 15); policy and guardrails (Ch 16).
- **Punctuation:** no em-dashes; a couple of ellipses. Antithesis kept out of explanatory body. Rungs held to 2-3 sentences.
