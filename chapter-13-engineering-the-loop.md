# Chapter 13: Engineering the Loop

*Draft in the author's voice. Part III. REWRITTEN per author: the chapter is about designing the loop at each scale (one agent, sub-agents, multi-agent) and how the Ch 11 shapes and Ch 12 topologies nest and combine. The previous guardrails-catalog draft is stashed at `.chapter-13-guardrails-draft.bak`; the parts of it that survive are folded into "What nesting costs." Running-example is a TODO block. In the wild leads with Google. No em-dashes; a few ellipses. Which phase: the loop.*

---

## Where is the loop?

By now you have drawn a lot of boxes. Shapes that put a model at the edge or the core, topologies that put whole agents in a row or under a supervisor. Here is the question none of those diagrams answer: where is the loop?

It matters because the loop is the thing that actually runs. A shape tells you what the pieces are; the loop tells you what happens on Tuesday at four in the morning, over and over, until something stops it. And once you have more than one agent, you do not have a loop. You have several, running inside each other, and the interesting design work is deciding which one is in charge.

This chapter walks up the scales: one agent, one agent with helpers, many agents. The rule that makes all of it tractable arrives in the second section, and everything after is a consequence of it.

## The loop in one agent

Start with the unit everything else is built from.

One agent is one loop. It reasons about what to do, does it, looks at what came back, and decides whether to go again. Around that sit exactly one of everything: one context that accumulates, one set of tools, one budget, one exit condition, one trace. When the loop ends, the agent is done.

**The tradeoff.** You get the cheapest, most debuggable thing in this book, and you are capped by one context window and one toolset. There is no parallelism and no specialization: a single agent reads twenty documents one after another, and it carries every tool it might ever need in front of it. You will know when you have outgrown it, because the failure is specific: the context fills up, and quality falls off a cliff that has nothing to do with the model.

## The loop with sub-agents

Now the agent delegates. It calls a sub-agent, and that sub-agent is a real agent, so it has a loop of its own. Two loops are running, one inside the other.

**To the parent, the entire inner loop is one step.** The parent calls, waits, and gets a result back. It never sees the sub-agent's fifteen turns, its false starts, or its retries. From where the parent sits, delegating looks exactly like calling a slow tool.

This is the Supervisor shape, and most frameworks cap it at one level deep.

Three things nest along with the loop.

**Budgets nest, and they multiply.** An outer loop of ten turns, each delegating to an inner loop of ten turns, is a hundred model calls. Allocate the budget downward as an explicit share rather than giving the sub-agent its own allowance, or the outer cap means nothing.

**Termination nests.** The inner loop needs its own exit, and when it trips, the parent needs to hear something useful. An inner loop that hits its cap has produced a failed step, which the parent reads as an observation and decides about. An inner failure that propagates up and kills the run throws away everything the parent had achieved.

**Traces nest.** The trace becomes a tree. That is mostly good: collapse a sub-agent's whole loop into one node and read the parent's story at the level it happened. It is bad the first time you need to know which of the sub-agent's fifteen turns produced the wrong summary, because the parent's trace shows only that the step succeeded.

**The tradeoff.** You buy context isolation, specialization, and a way to spend context that would never fit in one window. You pay in multiplied tokens, added latency, and a lossy boundary: the summary is all that survives, so a bad summary makes the parent confidently wrong with no way to tell. Two costs surprise people. The orchestrator accumulates context from every worker it calls, so at four or more workers it frequently blows its own window, which is the problem you delegated to avoid. And over enough turns the orchestrator's plan drifts from what was originally asked, especially when the request was vague to begin with.

## The loop across many agents

Now take away the orchestrator. Several agents, each with its own loop, and no single one in charge. Every question from the last section gets harder, and one gets urgent: **what ends the loop?**

With one agent, the agent finishes. With sub-agents, the orchestrator decides. Here, nothing decides unless you build the thing that does, and each topology from the last chapter answers it differently.

**Fan-Out ends when something consolidates.** Branches run in parallel, and the whole shape hinges on the join, which is the part people design last. You need a reducer: something that waits, merges, and handles the awkward cases. What happens when three branches return and the fourth hangs? What happens when two branches disagree? Sometimes the reducer is a few lines of code, and sometimes it has to be an agent in its own right, with its own loop, reconciling conflicting answers. Note what that means for timing: the parent's step does not finish until the join finishes, so the slowest branch sets the pace and one hung branch stalls everything.

**Debate ends when the judge says so.** The rounds are a loop with no natural exit, because two agents critiquing each other can always find one more thing to say. The judge is the exit, and it needs a bounded round count behind it. Take the judge away and you have oscillation with better manners.

**Peer-to-Peer does not end.** Each agent has its own exit condition, and the system has none, because no agent has the standing to declare the work finished. That is the same failure as the last chapter's, seen from the loop's side. The agents will keep waking each other up, and the thing that eventually stops them is your rate limit.

**The tradeoff.** You buy independence, and genuine parallelism, and the ability to run work that no single window could hold. You pay by having to build, by hand, everything the orchestrator was giving you for free: the exit, the consolidation, the accountability for whether the goal was met. Ask what ends the loop before you draw the second box, because the answer at this scale is never "it just does."

## Combining the shapes

Once an inner loop collapses to a step, the shapes compose freely, because any shape can be a step inside any other shape. That is the whole grammar.

A **Chain whose third step is an Autonomous Loop**: the pipeline is fixed and readable, and the one genuinely open part gets room to move, fenced inside a single step with its own budget and exit. This is the most common real architecture and the one to reach for first.

A **Supervisor whose workers are Chains**: the orchestrator decides which worker to call, and the worker does not improvise, because its shape is already decided. Judgment at the top, predictability underneath.

An **Evaluator-Optimizer wrapping a Fan-Out**: the generator step fans out across sources, joins, and produces a draft, and the critic loop decides whether to go again. Two loops, both bounded, the inner one invisible to the outer.

A **Router in front of everything**, dispatching to whichever of these fits the request. The router does not loop at all.

Combining is where the two rules meet. An inner shape is one step to its parent, and the only thing that crosses the boundary is a result written to a key the parent reads. Everything else stays inside. Which means the question to ask of any combination is the same one, at each level: what ends this loop, and who consolidates what comes back?

## TODO: running example

> **TODO (author):** the running-example section goes here, to be written once the running example is settled.
>
> *What this section should do:* Walk the running example up the scales, since that is the chapter's arc. Start with the single loop it began as: one context, one budget, one exit. Then delegate the deep log analysis to a sub-agent and show the collapse concretely, the sub-agent burns twenty turns reading logs and returns two sentences, and the parent's loop sees one step. Use it to make the budget-multiplies point real: ten outer turns each delegating twenty inner turns is two hundred model calls at 3 a.m. Then show the composition it settles into: a Router at the front (no loop), dispatching to a Chain per incident category (no outer loop), whose diagnosis step is a bounded Autonomous Loop (one driving loop), which Fan-Outs across services (parallel inner loops). Point at the driving loop at every level. Close on what crosses each boundary: a result written to a key, and nothing else.

## In the wild

Google's ADK is built on exactly this grammar. Its workflow agents compose by containment: a `SequentialAgent` holds a `LoopAgent` holds an `LlmAgent`, and because each one is an agent, each nests inside the next with no special case. Its own examples run a sequential pipeline whose middle stage loops until a sub-agent signals completion, which is the Chain-with-an-Autonomous-Loop-inside from a few paragraphs ago, shipped as a constructor argument. ADK is also explicit about where the driving loop sits: the workflow agents are deterministic and own the control flow, while the model-driven agents inside them do the reasoning, so "which loop is in charge" is readable from the structure rather than inferred from a prompt.

The other frameworks land in the same place from different directions, and the agreement on the sub-agent scale is the striking part. Anthropic's agent SDK ships supervisor as its native shape and caps subagents at one level deep, which is the nesting rule enforced by the API rather than left to your judgment. OpenAI's SDK does it through handoffs, LangGraph makes a whole graph usable as a node inside another graph, and every one of them treats fan-out as parallel dispatch that has to rejoin.

A published study of multi-agent financial document processing fans out extraction across independent branches and then needs a dedicated reconciliation agent to merge them, because the branches return conflicting readings of overlapping sections, and no amount of parallelism decides which one is right. Its own accounting is honest about the cost: latency is set by the slowest branch plus the merge. On the termination side, the durable-execution engines make the exit an explicit condition rather than a hope, checking the agent's own done flag *and* an iteration cap, and running compensation when a run dies after it has already sent the email or charged the card. Everyone who has run these things at scale converges on the same two questions this chapter asks: what ends the loop, and who consolidates.

## When not to reach for this

Nest when the work forces you, which is rarer than it looks. A single loop with good tools handles more than most people expect, and it costs one budget, one trace, and one exit condition. The moment you nest, you are paying multiplied tokens, added latency, and a debugging story that gets worse per level, so the question to ask of every sub-agent is whether a tool would have done it. Usually one would.

## Which phase

This is the **loop**, viewed as an architecture rather than a mechanism. The shapes compose because an inner loop collapses to a step, and the topologies differ because of where the driving loop sits. Everything after this builds on top of a loop you can point at.

---

### Author notes (not for the reader)
- **Running example (standing decision):** running-example section is a TODO placeholder pending the author's decision on the example.
- **Full rewrite (author's call):** the previous draft was a guardrails catalog (Monotonic Exit, Repetition Detector, Self-Check, Circuit Breaker, Budget, and so on) and the author rejected the whole premise: this chapter is about **designing the loop at each scale**, one agent, sub-agents, multi-agent, and how the Ch 11 shapes and Ch 12 topologies nest and combine. Old draft stashed at `.chapter-13-guardrails-draft.bak` in case any of it is wanted elsewhere. The parts worth keeping (budget multiplication, termination at boundaries, trace trees) survive as **What nesting costs**, reframed as consequences of nesting rather than a standalone catalog. Ch 10 already carries termination basics, so nothing is lost by dropping the catalog.
- **The load-bearing idea:** **an inner loop collapses into one step in the outer loop.** Stated once, in "The loop with sub-agents," and everything after is a consequence: budgets nest and multiply, termination nests (an inner cap produces a failed step, not a dead run), traces become trees, and the shapes compose freely because any shape can be a step inside any other shape.
- **Taxonomy correction (author):** the Supervisor topology was originally placed with the many-agent shapes; it belongs in **sub-agents**, because it has a clear orchestrator and is therefore just the nesting picture repeated. Confirmed by the frameworks: Anthropic's agent SDK ships supervisor natively with subagents capped at **one level deep**. The many-agent section is now reframed around the author's actual question, **what ends the loop when nothing is in charge**: Fan-Out ends when something consolidates (the reducer is the part designed last; partial failures, disagreement, and in at least one production study the reducer must itself be an agent), Debate ends when the judge says so (bounded rounds behind it), Peer-to-Peer does not end (each agent has an exit, the system has none, and the thing that stops it is your rate limit).
- **Tradeoffs per scale (author's request):** each of the three scales now closes with a bold **The tradeoff.** One agent: cheapest and most debuggable, capped by one window and one toolset, no parallelism or specialization, and the failure is specific (the context fills and quality falls off a cliff unrelated to the model). Sub-agents: buys isolation and specialization, pays multiplied tokens, added latency, and a lossy summary boundary; two costs that surprise people, the **orchestrator accumulates context from every worker and at four or more frequently blows its own window** (the problem you delegated to avoid), and **goal drift** as the plan diverges from a vague original request. Many agents: buys independence and real parallelism, pays by having to hand-build the exit, the consolidation, and the accountability the orchestrator gave you free.
- **The second rule:** **at every level, exactly one loop drives.** It is what actually distinguishes the Ch 12 topologies from each other (Supervisor has one; Pipeline has none; Fan-Out has one plus parallel inners; Debate has one plus a bounded inner ended by the judge; Peer-to-Peer has none and everyone thinks they have one). "If you cannot point at it, you do not have a design."
- **Sources (dated bucket, verify before print):** orchestrator-context-overflow at 4+ workers and **goal drift** from beam.ai multi-agent patterns (Jul 2026). Fan-out needs a **reducer** to merge outputs and handle partial failures (jobsbyculture, May 2026); the **dedicated reconciliation agent** resolving conflicting extractions from overlapping context windows, with latency set by slowest-branch-plus-merge, from the multi-agent financial document processing study (arxiv 2603.22651). Supervisor = **subagents one level deep** in Anthropic's agent SDK, plus the framework support matrix (LangGraph native fan-out/pipeline/supervisor; OpenAI SDK handoffs), from digitalapplied (May 2026). Durable-execution loop condition checking the done flag **and** an iteration cap, with compensation after real-world actions, from Conductor OSS docs. ADK workflow-agent containment (`SequentialAgent` / `LoopAgent` / `LlmAgent` nesting; workflow agents deterministic while the LLM agents inside them reason; loop terminates on `max_iterations` or a sub-agent escalation) from ADK docs. LangGraph graph-as-node. Coding agents spawning a fresh inner loop per file or failing test is well attested but keep it generic per the author's earlier direction on naming.
- **Cross-refs:** the loop as the unit + termination basics (Ch 10), the shapes (Ch 11), the topologies + the state rule (Ch 12), self-evolving loops (Ch 14).
- **Punctuation:** no em-dashes; a couple of ellipses. Antithesis kept out of explanatory body. Patterns held to 2-3 sentences per the author's rule.
