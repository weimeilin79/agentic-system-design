# Chapter 6: Memory Patterns

*Draft in the author's voice. Part II pattern chapter. Same catalog format as Chapter 5: voiced framing, the taxonomy, then named patterns with intent lines. No em-dashes. Which phase: context and harness (memory is what the agent knows, run by the runtime).*

---

## Copy, paste, start over

Think back to the early chat interfaces. Every session was a blank slate. You'd have a long, productive conversation, work something out together, close the tab, and the next day it was gone. To pick up where you left off, you did what everyone did: scroll up, copy the whole previous conversation, and paste it back in at the top of a new one. Or re-type the gist from memory. Or just give up and start over, re-explaining everything from scratch.

I did that more times than I want to admit. It was tedious and error-prone, you'd always forget the one detail that turned out to matter, and it put the entire burden of carrying context on the human. The tool remembered nothing, so you were its memory.

Memory is what ends that. It's what lets an agent recognize a returning user, pick up last week's decision, and stop solving the same problem for the first time every time, all without anyone copy-pasting a transcript. Its real job, underneath all of that, is to serve context: to store what matters so the right thing can be selected back into the window when it counts. In Chapter 2 memory was one of the five parts of the body; in Chapter 5, Externalize kept writing state out toward it and that `gotcha.md` hinted at it, every time context needed to outlive a single turn. This chapter is the real treatment.

But a warning before we start, because memory is also where you can do the most lasting damage. A bad fact in the context window rots one run. A bad fact written into long-term memory rots every run after it, quietly, until you find it. Memory multiplies whatever you put in it, the good and the wrong. So the patterns here are as much about forgetting and correcting as remembering.

## Isn't this just context?

Fair question, and here's the cleanest way to hold the two apart. **Context is selection: deciding what should be in the window right now.** Memory is storage: **how to keep things efficiently so you can put the right thing in the window later.** Memory exists to serve context. It's the pantry that makes the meal possible; context engineering is choosing what actually goes on the plate.

That's why they share a mechanism, retrieval, but not a job. Chapter 5 was the selection problem: of everything you *could* show the model, what earns a place in a scarce window. This chapter is the storage problem: how to hold everything you might later need, in a form that lets you find and return the right piece fast, cheap, and true. Do the storage well and selection gets easy; do it badly and no amount of clever prompting saves you.

Externalize is the hinge between them, context's hand reaching into storage, which is why it lived in the last chapter and points straight at this one. And the taxonomy below already says the same thing: working memory just *is* the window, selection's territory, and the other three types are the store that feeds it. Keep the two ideas distinct, because the failure modes differ. "The window rotted at turn forty" is a selection problem. "It remembered last quarter's owner" is a storage problem.

## The four types, and where they live

Two lenses, and keeping them separate saves you a lot of confusion. The first is *what kind of knowledge* you're holding. The second is *where it physically lives*. They're independent: the same fact might sit in the context window today and a database tomorrow.

The field has converged on four kinds of knowledge, drawn straight from cognitive science and formalized for agents by Princeton's CoALA framework. Every serious memory system builds on these:

- **Working memory:** the here and now, the current context window and scratchpad. This turn's material. We spent all of Chapter 5 on it.
- **Episodic memory:** specific past events. "On March 14th, this alert was caused by X and fixed by Y." Time-stamped, instance-level, the raw log of what happened.
- **Semantic memory:** general facts and preferences. "The payments service is owned by team Falcon." Not tied to a single event, just true.
- **Procedural memory:** how to do things. Learned runbooks, skills, the agent's own instructions and decision logic. Muscle memory.

The most useful thing about this taxonomy is the *movement* between the types, which we'll come back to: a repeated episode ("this alert keeps happening") gets promoted into a semantic fact, and then into a procedure. That promotion is where an agent stops just accumulating and starts getting wiser.

Where these live is the second lens: a small hot tier always in the context window, a warm retrievable store (usually a vector database), a structured store (a key-value store or a knowledge graph) when precision or relationships matter, and plain files for the rest. Choosing the store per type is the actual engineering.

## Memory patterns

Under a storage lens the patterns split three ways: the *store* it lives in, the *pipeline* it flows through, and the two *axes* that govern that pipeline. Get the store right first, because it decides what the pipeline can even do.

### The store

So where does memory physically sit? Two decisions settle it: how it's organized, and which store holds it. The first shapes the second, so start there.

#### How it's organized

**Tiered Memory** is the dominant idea, borrowed straight from how an operating system juggles RAM and disk. A small **core** tier lives always in the context window, the user and the current focus, instant to read but tiny and expensive per token. A **recall** tier holds recent history, searchable but not always loaded. An **archival** tier holds effectively unlimited history in external storage, fetched only on demand.

**The Temporal Knowledge Graph** organizes memory as a graph of entities and relationships, with every fact stamped by when it was true. It's the shape you reach for when relationships and time are the actual question, "who owned this service then, and has it changed since," which a flat pile of text simply can't answer. Zep and Graphiti are the reference implementations, and it's the one shape here that needs a purpose-built store.

**Scoping** is less a structure than a rule you enforce everywhere: every memory is namespaced by who it belongs to, user, session, agent, or organization. It sounds minor and isn't. Memory that bleeds from one user to another isn't a bug you file, it's an incident you disclose, so you decide the scope the moment you write, in whatever store you pick.

#### Choosing the store

The clean way to choose is to name the retrieval first: ask how you'll get the memory back, and the store mostly picks itself.

**Files** (markdown or JSON on disk) are the simplest and most common way to externalize memory, and they win when it's small, curated, and human-editable, retrieved by loading a document by name. Trivially written and edited by human or agent, version-controlled, portable, no infrastructure to run, searched by literal keyword. Cheap, and it gets you further than you'd expect, but no semantic similarity, no structured query, and no path to millions of entries. This is the skill-file and scratchpad case, and a fine home for the core and recall tiers.

**Key-value** wins when you retrieve by a known key: "give me user 12345's preferences." Fastest and simplest, no search at all, which makes it the natural home for scoped memory and, like files, for the hot tiers where you always know what you're fetching.

**A relational store (RDBMS)** wins when you need structured filters, joins, transactions, and consistency: "every unresolved incident for the payments service in the last thirty days." If your memory has fields you'll query and combine, and you care about getting the same answer twice, this is it, and it doubles as the safe system of record underneath everything else. What it won't give you is similarity over free text.

**A semantic (vector) store** wins when you need similarity over unstructured text: "find past conversations like this one," where there's no exact key and no clean field, only meaning. This is the home of episodic recall and anything RAG-shaped, and the usual choice for the archival tier. Its weakness is the flat namespace: no real sense of time, no relationships, and a talent for confidently returning stale matches when the index goes unmaintained.

**A graph store** is what the Temporal Knowledge Graph actually runs on, for when relationships and "as of when" are worth a purpose-built database. Powerful, and slower and harder to write than the rest.

Two things follow. Pick the wrong format and half the pipeline simply won't run: you can't Score and Retrieve by similarity out of plain files, and you can't ask a vector store a temporal question. And you'll almost never pick just one. The tiers make that concrete: a file or key-value core, a vector archival tier, sometimes a relational system of record, sometimes a graph beside them, each holding the memory whose access pattern it serves best. Tiered Memory isn't a store, it's this combination, wired together.

### The pipeline

A memory flows through three stages: **capture** (getting it in), **recall** (getting it out), and **maintenance** (keeping it healthy). In production these blur into one or two background jobs, which is exactly why they *feel* like the same operation. They aren't. Each step below does one small thing, and naming them apart is how you debug the one that's broken.

**Capture.**

**Extract.** *Distill the few salient facts from the raw interaction; never store the transcript.* A conversation is mostly noise. Pull out the handful of things worth keeping, a preference, a decision, an error and its fix, and drop the rest.

**Consolidate.** *Merge a new memory against what's already stored instead of blindly appending.* Before writing, check for near-duplicates and fold them together, so you don't accumulate five slightly different versions of the same fact.

**Reconcile.** *When a new fact contradicts an old one, supersede the old; don't keep both.* The classic rot: a note says PostgreSQL, a newer one says MySQL, and both sit there fighting. Reconciliation marks the old fact superseded and promotes the new. Zep's validity windows (a `valid_to` when a fact is replaced, an `invalid_at` when it's contradicted) are this made durable.

**Recall.**

**Score and Retrieve.** *Rank memories on relevance, recency, and importance, not similarity alone.* A memory can be highly relevant and badly stale, or old but critical. The Generative Agents work scored each memory on all three signals, and that blend is still the right instinct: store each memory with its embedding, a timestamp, and an importance rating, then rank on the combined score. And here is where the store's format bites or blesses you: relevance needs the embeddings, recency needs the timestamps, importance needs the rating field. This step is only ever as good as the fields your format chose to keep, which is why the store comes first.

**Maintenance.**

**Reflect.** *Review many episodes and synthesize higher-level knowledge to store.* The additive move: three similar incidents become one durable runbook, episodic promoted to semantic to procedural. Generative Agents named it, Anthropic uses the pattern to ship the Dreaming feature in Claude Code, and it shows up in Google's Antigravity stack as "reflective memory." This is how an agent gets *wiser*, not just fuller.

**Prune.** *Expire, decay, or delete stale and superseded memories on purpose.* The subtractive move, and Reflect's exact opposite. Give memories a time to live, decay their scores as they age, delete what Reconcile superseded. Most systems forget far too little, and a store without pruning is a poisoning machine on a delay.

### Two axes over the pipeline

Every step above is governed by two choices that are not themselves steps.

**Control: programmatic or agent-managed.** *Decide who drives each step: your code, or the agent through memory tools.* Programmatic memory is deterministic and safe, the developer decides what happens and when. Agent-managed memory hands the model CRUD tools over its own store, which is flexible and autonomous but needs the same guardrails and evaluation as any powerful tool. You'll usually mix them: let the agent manage low-stakes notes, keep the high-stakes writes on a programmatic path.

**Timing: hot-path or background.** *Decide when each step runs: inline and blocking, or async and eventual.* Extract and Reconcile can run in the response path (fresh but slow) or in a background sweep (cheap but eventually consistent). Reflect and Prune almost always run in the background, and almost always *together*, one "dream" job that synthesizes and forgets in a single pass. That shared timing is exactly why Reflect and Prune feel like one operation in the wild, even though one creates knowledge and the other destroys it.

## Aegis remembers

Watch the incident agent grow a memory, and watch every piece of this chapter show up in one system.

Start with the store. Aegis holds all four types at once: **working** memory is the live incident in the context window, **episodic** is ten thousand past incidents and how each was resolved, **semantic** is service ownership and the dependency map, **procedural** is its runbooks. Those types land in different stores, exactly as the format rules predict. The past incidents sit in a vector archival tier it can search by similarity. The ownership map lives in a **Temporal Knowledge Graph**, because "who owns payments, and did it change since this alert last fired?" is a question a vector blob can't answer. The hot facts, the current incident and the touched service, ride in a small **Tiered** core. And all of it is **Scoped** per service and team, because one team's runbook surfacing for another is an incident, not a convenience.

Now the pipeline, which runs every time an incident closes. Aegis **Extracts** the two things worth keeping, root cause and the fix, out of a forty-minute transcript. It **Consolidates** them against similar past incidents so the same lesson isn't stored five times. It **Reconciles** what changed: when payments moved to a new team, the old ownership fact is superseded, not duplicated. On recall, a new alert **Scores and Retrieves** the fix that actually worked, weighted for recency so it doesn't resurface a resolution from a system that no longer exists. And overnight, one background "dream" job **Reflects** recurring incidents into fresh runbooks and **Prunes** the runbook for the decommissioned service in the same pass.

The axes decide the rest. Aegis runs mostly **agent-managed**, letting the model curate its own low-stakes notes, but it keeps ownership reconciliation on a **programmatic** path, because getting "who owns prod" wrong at 3 a.m. is not a call the model gets to improvise. And it runs the heavy work on **background** timing, off the hot path, so triage stays fast while the learning happens after hours.

Episodic becomes semantic becomes procedural, and the agent that handled a thousand incidents is genuinely better at the thousand-and-first.

## When not to reach for these

Most agents don't need most of this. A stateless, single-shot task, classify this, extract that, answer this one question, has nothing worth remembering, and bolting a memory system onto it buys you cost, latency, and a privacy surface for no benefit. Start stateless. Add memory only when the task genuinely spans turns or sessions, and even then, add the cheapest tier that solves the actual problem. A conversation that needs to remember the last few turns needs working memory and maybe recall, not a temporal knowledge graph. Reach for the graph, the reflection loop, and the self-editing tools when the agent's value truly compounds over time, and not a moment before.

## Which phase

Memory straddles two phases. What the agent *knows* is squarely the **context** phase, memory is where context comes from when the answer isn't in this turn's window. But the machinery that forms, stores, retrieves, consolidates, and forgets it, live and mid-task, runs in the **harness**. So memory is a context concern you implement as a harness responsibility, which is exactly why Chapter 3 put memory management in the harness and this chapter puts the strategy here.

---

### Author notes (not for the reader)
- **Catalog format (regrouped by process, your call):** the material splits into three groups. **The store** now leads with three short organizing concepts (under the subtitle *How it's organized*) (Tiered Memory, the Temporal Knowledge Graph, Scoping), then a store-format menu (files, key-value, RDBMS, vector, graph) chosen by naming the retrieval first, concepts-before-stores so each store reads as "which shape it serves" rather than an abstract list; the tiers map explicitly onto the stores (core/recall to files or key-value, archival to vector, the graph shape to a graph store). **The pipeline** breaks the old coarse patterns into atomic steps a memory actually flows through: capture (Extract, Consolidate, Reconcile), recall (Score and Retrieve), maintenance (Reflect, Prune). **Two axes** govern the whole pipeline: Control (programmatic vs agent-managed, the old Self-Editing) and Timing (hot-path vs background). This resolves the Reflect/Prune/Self-Editing overlap: Reflect and Prune are opposite maintenance moves that share a background job (hence they feel identical in the wild, stated explicitly), and Self-Editing was really the control axis, not a peer step. The old Memory Formation was split into Extract + Consolidate + Reconcile.
- **Format-first spine:** the store's format decides what the pipeline can do (Score and Retrieve depends on the fields the format keeps), stated up front and validated against Claude Code (markdown + grep, skips graph and scoring) vs Zep/Graphiti (temporal graph + hybrid scored retrieval).
- **Framing:** two lenses (kind of knowledge vs. where it lives) and the four CoALA types up front, then the pattern groups. The standalone failure-modes section and the per-pattern *Targets* annotations were both cut; the failure each pattern addresses now reads from the pattern's own description (bloat, staleness, contradiction, leakage are self-explanatory in context).
- **Patterns and sources (dated bucket):** Tiered Memory (MemGPT/Letta, OS-style core/recall/archival), the capture steps Extract/Consolidate/Reconcile (Mem0's extract-consolidate pipeline with conflict resolution; Zep's validity windows for Reconcile), Score and Retrieve (Generative Agents' recency+importance+relevance; hybrid BM25+embedding in Zep/memsearch), Reflect (Generative Agents reflection; Anthropic's Dreaming in Claude Code; "reflective memory" in Google Antigravity/Vertex), Prune (the known forgetting gap in most systems), the Control axis (MemGPT agent-managed vs programmatic), Temporal Knowledge Graph (Zep/Graphiti). Four-type taxonomy from Princeton's CoALA (2023). All current per searches at draft time; frameworks and product specifics are dated.
- **In the wild to weave/expand if wanted:** LangMem (hot-path vs background formation, procedural self-editing), Anthropic's "7 Layers of Memory" (March 2026), Spotify Ads AI (session store + in-memory cache + Postgres). Left light to keep the chapter proportionate.
- **Continuity:** builds on Chapter 5 and Chapter 2 (memory as a body part). Externalize stays in Chapter 5 but is framed as "externalize into memory," the write-bridge to this chapter; the raw Externalize dump is the crudest form of the capture pipeline (Extract/Consolidate/Reconcile) on this side of the seam. The consolidation/promotion thread (episodic to semantic to procedural) is the spine and pays off in Aegis. Prune explicitly calls back to Chapter 5's poisoning "machine on a delay."
- **Which phase:** context (what it knows) implemented as a harness responsibility (the runtime that runs the memory), consistent with Chapter 3.
- **Punctuation:** no em-dashes; a few ellipses. Pattern names bold, intents italic.
