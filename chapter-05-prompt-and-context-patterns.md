# Chapter 5: Prompt and Context Patterns

*Draft in the author's voice. Part II pattern chapter, covering the prompt and context phases from Chapter 3 in depth. Register: voiced framing prose, named patterns as the backbone. No em-dashes. Which phase: prompt and context (the runtime work hands off to the harness).*

---

## More is worse

Every instinct you have about context is wrong. I say that kindly, because mine were too.

When an agent gives a bad answer, the natural move is to give it more. More instructions. More examples. More documents. More history. A bigger window must mean a smarter agent. It's such a reasonable assumption, and it's backwards. Past a point, more context makes an agent *worse*, not better, and developing a feel for where that point sits is the most useful skill in this chapter.

In Chapter 3 I told you the prompt is the phase we've mostly solved and context is where the real leverage is. This chapter is the how. And the first thing to internalize is that prompt and context aren't two separate crafts anymore. They're one job: deciding what the model sees on every single call. Put plainly, context engineering is *selection*, choosing the few things that earn a place in a scarce window. The wording of the instruction is part of it. So is every document, every tool result, every scrap of history that shares the window with it. All of it competes for the same thing.

## The finite attention budget

Here's the mental model that makes the rest click. A model has a *finite attention budget*. Every token you put in the window competes for that attention, and as the window fills, precision drops, reasoning weakens, and the model starts missing things it would have caught in a cleaner context. The field has a name for the failure now: **context rot**. Quality degrades as the window grows, and not gently. Past a certain fraction of the maximum, accuracy can fall off a cliff rather than a ramp.

The seductive lie of the last few years was that giant context windows would make this go away. They didn't. A million-token window doesn't solve context rot, it just gives you more rope. There's even a well-documented "lost in the middle" effect: models pay the most attention to the start and end of a long context and quietly skim the middle, so burying the one fact that matters in the center of a huge dump is a good way to have it ignored.

So the goal is never a full window. The goal is the *smallest set of high-signal tokens* that gets the job done. Everything that follows is a named pattern for getting there.

## The four ways context breaks

Patterns solve recurring problems, so name the problems first. Context fails in four recognizable ways, and the patterns in this chapter are responses to them.

- **Poisoning:** a wrong fact enters context early, gets treated as truth, and compounds through later steps.
- **Distraction:** too much history, and the agent fixates on something irrelevant instead of the task at hand.
- **Confusion:** conflicting retrieved documents, and the model can't tell which one is right.
- **Clash:** new information contradicts what's already in the window, and both degrade everything around them.

When you see an agent misbehave, you'll usually be able to name which one is happening. That's the tell for which pattern to reach for.

## Prompt patterns

The prompt is the mature phase, so this is a short shelf. Three patterns carry most of it.

**The Structured Prompt.** *Give the model a consistent skeleton at the right altitude, and version it as code.* Prose prompts drift and rot. Give every prompt the same bones instead: role, task, constraints, and the shape of the output. Keep each part at the right altitude, specific enough to be useful, general enough that the model can adapt. Too rigid and you've hard-coded brittle logic; too vague and you've said nothing. Then treat the whole thing as code: version it, and keep a suite of fixed test cases you re-run every time you touch it. A prompt you can't test is a prompt you're guessing at.

**The Output Contract.** *State the output format in the prompt, and enforce it at the parse layer.* The model will eventually hand you something that doesn't match what you asked for. So defend in depth: declare the exact output shape in the prompt, then validate it when it comes back, repairing or rejecting anything that doesn't conform. Never trust the prompt alone. A parser that assumes clean model output is a 3 a.m. incident waiting to happen.

**Exemplars.** *Show the behavior you want, don't just describe it.* Description has limits; demonstration is precise. When a task has a particular shape, a format to match, an edge case to handle, a tone to hit, put a couple of worked examples right in the prompt. A model pattern-matches off two good exemplars far more reliably than off a paragraph trying to describe the same thing. Use the fewest that pin the behavior, because each one costs tokens.

## Context patterns

Now the real catalog. Each pattern answers a specific failure mode, and mature agents use several together. One principle sits over all of them, and it fights every instinct you have: **curate, don't accumulate**. Part of that curation is semantic nuance, the quiet killer. If the agent doesn't know that "active users" means one thing to product and another to billing, it will be confidently, precisely wrong, and nothing in its fluent answer will tell you. Encoding what your words actually mean is context work retrieval alone won't do for you.

**Externalize into memory.** *When the window fills, write working state out into memory and carry a reference instead of the payload.* The message thread grows monotonically. Every tool result and intermediate finding piles up until it rots the window. So write that material out into the store, a scratchpad, a file, the memory tiers of the next chapter, and keep only a lightweight pointer in-band, pulling the full content back when it's needed. This is the seam between context and memory: Externalize is the *write*, and memory (Chapter 6) is where it lands. It's also the foundation the reduce-and-retrieve patterns lean on, because you can only pull back what you first pushed out. *Targets distraction and bloat.*

**Just-in-Time Retrieval.** *Pull the specific thing you need, when you need it, via lightweight identifiers, and rank hard.* Front-loading everything relevant is how you drown a model in noise. Instead, hold references (file paths, IDs, query strings) and retrieve the actual content on demand. This is classic retrieval plus its harder half, relevance: bad retrieval is one of the largest sources of hallucination in production, so real systems use several ranking signals and rerank aggressively, because pulling ten mediocre documents is worse than pulling the two that matter. Think card catalog, not library. And keep the index fresh, because a stale one quietly serves last quarter's truth and becomes a poisoning machine. *Targets confusion and poisoning.*

**Compaction.** *Summarize accumulated history into a dense signal before the window rots.* On a long task, even well-selected context piles up. So periodically summarize older turns and verbose tool outputs into something dense, keeping the decisions and dropping the dead ends. Good systems run a compaction pass between major steps rather than waiting to overflow. This is common practice among the top-tier agent harness tools, which auto-compact as the window nears its limit, preserving architectural decisions and open threads. The risk to respect is *context collapse*: over-compress, and repeated rewriting erodes the very details that made the context useful. Summarize deliberately, not constantly. *Targets rot and bloat.*

**Context Isolation.** *Give each subtask its own clean window.* Some work doesn't belong in one context. Hand a subtask to a sub-agent or a separate process with its own focused window, let it work, and return only the result. A research step that reads twenty documents can run in a subagent whose context never pollutes the main thread; a risky tool call can run in a sandbox whose state stays out of the model's window until it's needed. This is the bridge to the multi-agent patterns in Part III, and the reason "when agents work together" from Chapter 2 exists. *Targets distraction and scale.*

**Cacheable Prefix.** *Order the prompt so the stable part gets cached and only the volatile tail is recomputed.* This one is cross-cutting and pure economics, and teams underrate it badly. An agent resends most of the same context every step, so structure it deliberately: stable material first (system prompt, tools, durable context), volatile material last (the latest turn), so provider prompt caching charges you a fraction for the unchanged prefix. Two rules make or break the hit rate: keep the prefix byte-for-byte stable (a timestamp precise to the second at the top of your system prompt will quietly destroy it), and keep context append-only, never editing past turns, with deterministic serialization. This isn't provider-specific folklore. Every frontier provider offers prefix caching, and the frameworks bake it in: Google's ADK, for one, ships a `static_instruction` primitive whose whole job is keeping the cache prefix immutable across calls, plus a `ContextCacheConfig` to tune it. Turning caching on, and structuring context so it actually hits, is often a bigger win than any model change. *Targets cost and latency, on every call.*

**Recitation.** *Keep the goal in the model's recent attention by restating it.* On a long, multi-step task the agent drifts, and thanks to lost-in-the-middle the original objective, stated way back at the start, is exactly where attention has gone weakest. So have the agent keep a running plan, a todo list, an objective block, and rewrite it near the *end* of the context every few steps, checking off what's done. Restating the goal pushes it back into the most-attended region of the window. Some agents keep a `todo.md` they constantly rewrite for exactly this. Note how this differs from Compaction: compaction shrinks the past, recitation re-surfaces the goal. *Targets distraction and drift.*

**Keep the Errors.** *Leave failures in context so the model learns from them.* The instinct is to scrub failed actions and stack traces out to keep the context clean. Resist it. When the agent sees its own missteps, the failed call and the error it returned, it adapts and stops repeating them; hide them and you've deleted the evidence it needed to recover. Some production teams leave the wrong turns in on purpose. This sounds like it contradicts the poisoning warning, and the reconciliation is the whole point: a genuine *error* is useful signal and should stay, but a hallucinated *fact* is poison and must not be allowed to harden into truth. Keep the failures visible; guard against the false assertions. There's also a cross-session cousin worth knowing: a `gotcha.md` the agent writes recurring lessons into and reloads next time, which is Externalize aimed at mistakes, written into memory (more in the memory chapter). *Targets error recovery, the deliberate counterweight to over-scrubbing.*

## Package it: skills

One more, because it's how the patterns above scale past a single clever prompt. Rather than re-explaining a domain to the agent every time, capture the essence of a role or a domain once, in a file the agent loads: an `agent.md`-style setup, a per-domain skill, a reusable block that says "here is how we do this kind of work." It's Externalize aimed at knowledge instead of a single run's state, written once into memory and reloaded when needed (Chapter 6). This is exactly what teams like Spotify described when they ran a coding agent across thousands of repositories. It's all about reproducible, well-curated context, packaged and reused. And one habit ties everything together: test it, or you're guessing. Keep a suite of fixed tasks, run it after every change to a prompt, a retrieval rule, or a compaction setting, and watch window utilization, retrieval relevance, and cost per task. Context engineering without an evaluation loop is just vibes, and vibes don't survive production. Evaluation gets its own chapter later; it starts here.

## Aegis, patterned

Watch the incident agent assemble itself out of these.

Its prompts are **Structured** (a per-service skeleton) with an **Output Contract** on the incident record and a couple of **Exemplars** of good root-cause writeups. It **Externalizes** its investigation notes into memory rather than dragging forty minutes of logs through every step. It uses **Just-in-Time Retrieval** to pull only the runbook matching the alert type, off a freshly maintained index. It **Compacts** the investigation as it grows, keeping findings and dropping dead ends. It **Isolates** a deep log-analysis step into a subagent whose thousand lines of output never touch the main window. It orders everything behind a **Cacheable Prefix**, because at 3 a.m. across a hundred alerts, the token bill is real. It **Recites** its current hypothesis and remaining checks each step, so on minute forty it hasn't lost the thread of what it set out to prove. And it **Keeps the Errors** in: a remediation that failed stays visible so it doesn't retry the same dead end. That last one is exactly where the poisoning guard earns its keep. Keep the failed *attempts* in view, but never let a wrong *guess* about root cause harden into fact by the tenth step.

## When not to reach for these

These patterns are for agents that run long and handle a lot of context. A short, single-shot task with a small, stable set of information needs almost none of them. Put the fixed context in a Structured Prompt with an Output Contract and stop. Don't build retrieval when the whole knowledge base fits in the prompt, don't compact a five-turn conversation, don't isolate work that was never going to pollute anything. Every pattern here is also a dependency and a failure mode. The craft is spending your attention budget wisely, and sometimes the wisest spend is not building the machinery at all.

## Which phase

This is the **prompt and context** phases, the first two, and it's where most teams already spend their effort, so the message is subtraction, not addition. But watch the handoff: deciding *what* the agent sees is context engineering. The patterns are yours to choose. The runtime that executes them, live and mid-task, is the harness, the next phase.

---

### Author notes (not for the reader)
- **Merge and cleanup:** flowing prose framing (more is worse, finite attention budget, the four failure modes, curate-don't-accumulate, semantic nuance, skills, the eval-loop habit) plus the named patterns as the backbone.
- **TODOs resolved:** (1) Cacheable Prefix now states plainly this isn't Manus-specific: every frontier provider offers prefix caching, and ADK has first-class support (`static_instruction` primitive, `ContextCacheConfig`); closing payoff and Targets line restored. (2) Keep the Errors distinguishes the in-run trace (the pattern) from the cross-session `gotcha.md` cousin (Externalize into memory aimed at mistakes, flagged for the memory chapter).
- **Consistency fixes:** unified the pattern name to "Just-in-Time Retrieval" (ranking emphasized in intent and body rather than in the heading); anonymized the remaining vendor reference in Keep the Errors to match the anonymized examples elsewhere (attribution kept here in the notes); fixed "an agent.md," "the others build on," "Some agents keep a todo.md," "described when they ran," the stray space in "Compaction," and the curly apostrophe; named the harness in the Which-phase close so it no longer floats.
- **Sources (dated bucket):** the context patterns map to LangChain's Write/Select/Compress/Isolate; Externalize is kept here but framed as "externalize into memory," the write-bridge to Chapter 6, since the store it writes to is memory. Recitation, Keep the Errors, and the append-only/byte-stable Cacheable Prefix rules come from the Manus team (Yichao Ji). ADK caching (`static_instruction`, `ContextCacheConfig`) verified in ADK docs at draft time. ADK also natively does filtering, compaction, and artifact lazy-loading, so it can serve as an in-the-wild example for several patterns.
- **Punctuation:** no em-dashes; a few ellipses. Pattern names bold, intents italic.
