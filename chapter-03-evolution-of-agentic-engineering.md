# Chapter 3: The Evolution of Agentic Engineering

*Draft in the author's voice. One of the two chapters that carry the book (with the loop cluster in Chapters 11 to 13), so it runs voice-forward. It pays off the "four words" teaser from Chapters 1 and 2, reframed as four phases of the craft. Register: hot in the opener, the thesis, and the close; clean when defining each phase. No em-dashes.*

---

## Four words, four phases

This all happened fast. In about two years, the way we build agents has moved through four phases, and I've had a front-row seat for the whole run, wrong turns included. Each phase showed up looking like the answer. Each one turned out to be a layer of something bigger.

I promised you a mental model two chapters ago and kept calling it four words long. Here they are.

Prompt. Context. Harness. Loop.

The prompt is what you ask. The context is what the agent knows when you ask it. The harness is what it can actually *do* about it. And the loop is how it checks its own work and goes around again. The order those phases arrived in is not an accident. It tracks how much each one adds and how hard each one is, and it's roughly the order you should build in.

**The field moved through all four fast. The products didn't. A lot of what shipped early was built on just the first phase or two, and that gap is exactly why so many early agents feel flaky.**

Let me walk you through the phases, and be honest about where each one actually sits.

## The prompt phase

In the beginning was the prompt, and the prompt was treated as magic.

This was the first phase of the whole field. We minted "prompt engineer" as a job title. We traded incantations like trade secrets. "You are an expert SRE, think step by step, take a deep breath." For a while, the wording *was* the work, and it felt like the whole game was finding the right spell.

Here's the honest read, and it isn't the fashionable one. All four phases matter. But the prompt is the phase we've had the longest, which means it's the one we've mostly figured out. The patterns are well understood, the tooling is mature, and a clear, well-structured prompt is largely a solved craft. We still spend a chapter on it in Chapter 5, because "largely solved" is not "trivial." The point is that this is the phase where you're least likely to be the reason your agent is failing.

The trap isn't that the prompt doesn't matter. It's mistaking a good prompt for a finished agent. Nailing this phase feels like real progress, and it is, but it's the first step of four, not the whole staircase. The early wave of agent products got built mostly here, and you can feel it when you use them.

## The context phase

Then we noticed something uncomfortable. The same prompt, on the same model, gave one person a sharp answer and another person confident nonsense. The difference wasn't the wording. It was what the model was allowed to see.

That opened the second phase. Context engineering is the craft of what the agent has in front of it when it reasons: the relevant runbook, the recent deploys, the current state of the system, the meaning of the words you're using. Back in the anatomy, this was the senses. It's a bigger lever than the prompt, and the craft is the opposite of what people expect. It isn't stuffing everything you own into the window and hoping. It's the discipline of putting the *right* things in front of the agent at the right moment and keeping the noise out. Curation, not accumulation. Chapter 5 is this phase in full.

## The harness phase

Then a harder truth. An agent that can think and see but can't touch anything is just a very expensive opinion.

The harness is the entire runtime around the model. One useful definition doing the rounds is simply everything that isn't the model itself. That includes the tools it can call and the sandbox it runs them in, yes. But it also includes the machinery that manages the agent's memory across a long task: the short-term context window, the working scratchpad, the long-term store, and the constant decisions about what to pull in and what to compact out on every single step, so the model stays focused instead of drowning in its own history. It includes the state that survives a crash, so a long job resumes from a checkpoint instead of starting over. It includes the error recovery when a tool fails and the agent has to find another way. And it includes the orchestration that carries one step into the next. And when the work runs long, it includes spinning off background threads to keep watch over it, so a job that takes hours isn't fired and forgotten.

This is the runtime I promised you in Chapter 1, and it's where most of the real engineering turns out to live. Build it well and the whole system goes model-agnostic: the tools, the memory, and the logic sit in the harness, so swapping in next month's better model becomes a config change instead of a rewrite. The phase is so new the name barely predates this book: "harness engineering" only entered the vocabulary in early 2026, once enough people had built one to realize it was the actual job.

## The loop phase

And now the phase we're living through, the one most of the shipped products haven't reached yet.

Loop engineering is what happens *after* the agent acts. It looks at the result, decides whether it's good enough, and if it isn't, it goes again. Check, correct, repeat, until the task is truly done or until the agent knows it's stuck and says so. Verification. Retries. Evaluation gates. Knowing when to stop.

Here is the most important sentence in this chapter, so I'll keep it short. **Reliability does not live in the model. It lives in the loop.**

This is why the demo dazzles and the production system lets you down. A demo is one pass through these phases with a human standing by to catch the mistake. Production is the same run, unattended, at 3 a.m., and the only thing between a good outcome and a silent, confident error is whether you built the loop that catches it. This phase is so much of the real work that Part III of this book is devoted to almost nothing else.

## The weight is upside down

So here's the whole argument of the book in one picture.

The four phases arrived in order, and they get harder and higher-leverage as you go. Prompt is mature. Context is getting there. The harness (except tooling) and the loop are the new frontier, and they're where the reliability gap actually lives. Now look at where the effort actually goes. Most of the tooling, the tutorials, and the attention still pile onto the two phases we've had longest and mostly solved. The two that decide whether an agent survives production get the least. We've spent the most on the phases that need it least.

The early wave of agent products got shipped on exactly that imbalance. Strong prompts, decent context, good tooling ecosystem, a thin harness, no real loop. That's why so many of them impress for a minute and frustrate for an hour.

The fix is to stop treating a good prompt as a finished agent, and to put real engineering into the harness and the loop, the phases where trustworthiness actually comes from. That inversion is the thesis, and it's why this book spends one chapter on prompting and an entire part on the loop.

## TODO: running example

> **TODO (author):** the running-example section goes here, to be written once the running example is settled.
>
> *What this section should do:* Re-read the running example's v0 failures as phases it never reached, then walk it forward: senses (context), hands (tools), memory, the loop. This is the chapter's payoff, so it needs the v0 from Ch1 to line up.

## The evolution is the map

That's the evolution, and it's also the map for everything that follows.

Part II lives in the first three phases, applied to a single agent: its context, its memory, its harness. Part III is loop engineering, and everything that makes a loop trustworthy once there's real work and real risk on the line. Every chapter from here ends by telling you which of the four phases it belongs to, so you always know where you're standing.

I watched these phases arrive one at a time, and got a few of them wrong before I understood each was only a step. You don't have to take that route. That's what the map is for. You've got it now: prompt, context, harness, loop. Let's start building.

---

### Author notes (not for the reader)
- **Running example (standing decision):** the running-example section is a TODO placeholder pending the author's decision on the example itself. Original Aegis prose is preserved in `aegis-sections-stash.md`. Apply the same convention to all future chapters: draft the chapter, mark the running-example section as TODO with a note on what it should do.
- **Proportionality (your note):** the opener no longer implies a career of "living through eras." It says plainly this happened in about two years, four phases, fast. "Phase," not "era," throughout.
- **Fairness fix (your note):** dropped "most teams never left the prompt phase / the tragedy" (unfair and not real). Replaced with the accurate version: the field moved fast, the *products* didn't, and the early wave shipped on just the first phase or two, which is why early agents feel flaky. This shows up in the opener bold line, the prompt phase, and the thesis.
- **Prompt claim corrected (your note):** removed "the prompt is the least important." New claim: all four matter, but the prompt is the phase we've had longest and mostly figured out, so it's the one you're least likely to be failing on. The trap is mistaking a good prompt for a finished agent.
- **Harness expanded (your research request):** harness is now clearly more than tools. Added memory management (short/working/long-term, compaction, retrieval per step), state persistence across crashes, sandboxed execution, error recovery, and orchestration, using the current "everything that isn't the model" definition. Noted it makes systems model-agnostic and that the term is new (early 2026). Sources: MongoDB/LangChain harness series, Databricks, DataCamp, Firecrawl, the agent-harness arXiv chapter.
- **Thesis reframed:** "weight is upside down" now argues maturity is front-loaded and the reliability gap is in the newer phases (harness, loop), not "prompt doesn't matter / teams are stuck."
- **Kept:** "a very expensive opinion," "reliability does not live in the model, it lives in the loop," the Aegis failure-to-phase mapping, the map close.
- **Punctuation:** no em-dashes; a few ellipses. One bold turn per section.
