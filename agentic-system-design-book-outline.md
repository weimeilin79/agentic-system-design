# Agentic System Design
### Patterns for Engineers and Architects Building Autonomous Systems

*Full chapter-by-chapter outline / table of contents (working draft)*

---

## About this book

**Audience:** engineers and architects designing, building, and operating agentic systems — people who will write the harness, wire the tools, and get paged when the agent does something surprising.

**Promise:** by the end, the reader can look at any agent requirement and decompose it into the right patterns, know which parts to make deterministic, and design the loop that makes it reliable in production.

**Three devices carried through the whole book:**

1. **The four phases.** Every chapter hangs off one progression, **Prompt, Context, Harness, Loop**, the four phases the craft evolved through (Chapter 3). Each pattern chapter ends by naming which phase it belongs to.
2. **The running example — TODO (author to decide).** A single system built up across the book, so the reader watches a real agent accrete from a prompt into a governed production system. Each pattern chapter adds one capability to it. The working placeholder has been "Aegis," an incident-response agent (triages production alerts, reasons over logs and dashboards, consults runbooks, proposes and with approval executes remediations, learns from outcomes), and the per-chapter *Aegis:* bullets below are written against that placeholder. **They stand as sketches of what each running-example section must do; the example itself is not settled.** In the drafted chapters, these sections are marked as TODO blocks, and the original prose is preserved in `aegis-sections-stash.md`. Contrasting "big-name" case studies (Spotify Honk, ServiceNow, and others) appear alongside to show the same pattern at scale.
3. **"Design Debate" sidebars.** The major platforms have converged on architecture but openly disagree on philosophy — and those disagreements teach better than any single vendor's answer. Recurring one-page sidebars stage the five live debates at the chapter where each bites hardest, present both camps fairly (important given the author's Google affiliation — the book stays vendor-neutral), and leave the reader equipped to decide:
   - **Reach for multi-agent, or resist it?** (Google/Microsoft/AWS lean in; Anthropic/Cognition say start single-agent) → Chapter 12.
   - **How much of the loop to own?** (own the loop vs. govern the loop; a ladder every vendor sells end to end, from raw API to managed runtime) → Chapter 2, revisited Chapter 10.
   - **Where do guardrails live?** (IAM/policy vs. middleware pipeline vs. model/harness) → Chapter 16.
   - **Own-the-stack vs. framework-agnostic?** (Google's integrated bet vs. AWS/Microsoft "host any framework") → Chapter 18.
   - **Code-first vs. visual/no-code?** (ADK/Strands/Agents SDK vs. Agent Studio/AgentKit/Foundry) → Chapter 7.

**Chapter template (every pattern chapter follows it):**
1. *The opener* — a way in that **varies by chapter** (see the opener menu below). Reserve the dramatic *failure* opening for just 2–4 chapters so it keeps its punch.
2. *The pattern* — the core idea and vocabulary.
3. *Sub-patterns & trade-offs* — the real design decisions.
4. *Aegis* — applying it to the running example, with sketch-level code/config.
5. *In the wild* — how a shipped system does it.
6. *When not to use it* — the anti-pattern and cheaper alternatives.
7. *Which phase* — which of the four phases (prompt, context, harness, loop) this belongs to, and what it feeds.

**The opener menu (rotate these so no two adjacent chapters open the same way):**
- **The failure** *(2–4 chapters only)* — a concrete way agents break without this pattern. Save it for where the failure is genuinely visceral: **Policy and Guardrails (Ch 16)**, **Loop Validation (Ch 13)**, and one of **Evaluation (Ch 9)** or **The New Bottlenecks (Ch 22)**.
- **The scene/scar** — a lived moment from the trenches (a beginner who couldn't `cd`; a 2 a.m. page).
- **The reframe** — an antithesis metaphor that flips the reader's assumption (*a bridge, not a gate*).
- **The question** — a provocative question the chapter then earns the answer to.
- **The stakes** — why this pattern matters *now*, tied to where the field is.
- **Straight in** — for reference-heavy chapters, skip the flourish and define fast; a clean open is itself a palate cleanser between voiced chapters.

A good rhythm: never more than two "hot" openers in a row, and let the mechanics-heavy chapters (Memory, Tooling, Deterministic Workflows) open straight so the voiced ones land harder by contrast.

**Voice & register (see the companion Voice & Style Guide):**

The book is written in the author's voice — a veteran practitioner who has the scars, refuses to be dogmatic about any stack, and treats experience as *a bridge to bring the reader in, not a gate to keep them out*. That anti-dogma stance is also the book's vendor-neutral insurance policy. But memoir doesn't scale to 500 pages, so voice is assigned by **function, not evenly**:

- **Full voice (~65–80%)** — the book intro and close, part openers, each chapter's opener (whichever opener type it uses), the Design Debate sidebars, and the *"When not to use it"* reflections. These frame, persuade, and reflect; they carry one bold turning-line and short evocative headers. The full scar → tension → turn → generosity arc is reserved for the *failure* and *scene/scar* openers, not every chapter.
- **Clean register (~10–20%)** — the pattern mechanics, sub-patterns, code, tables, and Aegis config. The moment the reader needs to *implement or decide fast*, the prose gets precise and gets out of the way.

Rule of thumb: run the voice hot when framing; cool when the reader is doing. A one-page style guide locks the rhythm, header style, and signature moves so all 25 chapters read as the same person even when drafted out of order.

---

# Part I — Foundations

### Chapter 1. From Automation to Autonomy
- Opener: "the waves" (a veteran who has ridden many tech waves, anti-dogma) rather than a failure opener.
- Automation (you specify the steps) vs. autonomy (you specify the goal, the machine chooses the steps): the dividing line the whole book is built on.
- The reliability gap: capability racing ahead of oversight, and why most pilots stall on scaffolding, not intelligence.
- The engineer's reframe: you are building a **runtime**, not calling an API.
- *Aegis v0:* the naive "just prompt an LLM with the alert" version, and its five failures (no eyes, no hands, no memory, no loop, fails silently), each one a later chapter.

### Chapter 2. The Anatomy of an Agent
- The single agent's **anatomy** (five parts): brain (reasoning model), senses (context), hands and feet (tools), memory, and a voice (talking to other agents).
- **When agents work together:** the group-level concerns that come to the front once there is more than one agent, orchestration, identity and guardrails (a social contract: which tools, which agents it may talk to), and observability (built into each agent and aggregated across the group).
- Choosing the model ("feels agentic"): tool-calling behavior, instruction-following, graceful failure; design so the model is a swappable config change.
- Build our own, or reach for a framework: start high-level, peel abstraction away as the stakes rise.
- *Vendor lens:* the industry converged on the same anatomy (why the reference model is vendor-neutral); Anthropic's three principles and Ng's four patterns as the first-principles on-ramp.
- *Design Debate (how much of the loop to own):* own the loop vs. govern the loop, a ladder each vendor sells end to end (Gemini API, to ADK, to Antigravity SDK, to managed platform). Introduced here, revisited Chapter 10.

### Chapter 3. The Evolution of Agentic Engineering
- The book's core mental model: four phases the craft moved through in about two years, in the order of how much each adds and how you should build.
- **Prompt:** the phase we have had longest and mostly solved; the trap is mistaking a good prompt for a finished agent.
- **Context:** what the agent is allowed to see; curation, not accumulation.
- **Harness:** the whole runtime that is not the model (tools, memory management, state persistence, error recovery, orchestration, background monitoring of long jobs); where most of the engineering lives, and what makes the system model-agnostic.
- **Loop:** verification, retries, eval gates; reliability lives here, not in the model.
- The thesis: maturity is front-loaded on the early phases; the reliability gap is in the newer ones (the harness beyond tooling, and the loop). The early wave of products shipped on that imbalance and it shows.
- *Aegis:* v0's five failures re-read as phases it never reached; building Aegis is walking it forward through the phases.

---

# Part II — The Core Patterns

### Chapter 4. Multi-Model Patterns
- The core question: when to use which model. One model rarely fits a whole agent.
- Match the model to the task: a small, fast, cheap model for routing, classification, and simple steps; a frontier model for hard reasoning and planning; specialized models (code, vision, embeddings) where they win.
- **Model routing and cascades:** try the cheap model first, escalate to the expensive one only when the task or a confidence check demands it.
- **Build your own with fine-tuning:** when a fine-tuned open-weight model (Llama, Qwen, Mistral, GPT-OSS, and the like) beats a frontier API model, usually on a narrow, high-volume, latency-, or privacy-sensitive task. Techniques to name: parameter-efficient fine-tuning (LoRA/QLoRA) and distillation (train a small open model on a frontier model's outputs for most of the quality at a fraction of the cost). The trade: you now own serving, evaluation, and drift.
- The three-way tension: cost, latency, capability, and how per-step routing lets you tune it.
- **Fallbacks and resilience:** what happens when a model is down, rate-limited, or refuses; multi-provider redundancy.
- Staying model-agnostic (callback to Chapters 2 and 3): the harness makes swapping and routing a config change, not a rewrite.
- *Aegis:* a small fine-tuned open model triages and classifies the alert on-box; a frontier model does the root-cause reasoning; a fallback model when the primary is rate-limited at 3 a.m.
- *In the wild:* production stacks that put a cheap classifier in front to route, then specialized or stronger models behind it; harnesses that send simple steps to small models and hard steps to large ones.
- *When not to use it:* a single capable model is simpler; do not add routing until cost or latency forces it, and do not fine-tune unless the task is narrow, stable, and high-volume enough to repay the MLOps cost (a frontier API is the right default for broad, changing, or low-volume work).
- *Which phase:* harness (routing is a harness job), with a foot in the loop (escalation is a feedback decision).

### Chapter 5. Prompt and Context Patterns
- **Framing:** more is worse. A model has a *finite attention budget*; context rot degrades quality as the window fills, and giant windows don't fix it. Context engineering is **selection**: choosing the few things that earn a place in a scarce window.
- **The four ways context breaks:** poisoning, distraction, confusion, clash (each pattern targets one or more).
- **Prompt patterns (a short shelf, the mature phase):** the Structured Prompt (a consistent skeleton at the right altitude, versioned as code), the Output Contract (format in the prompt, enforced at the parse layer), Exemplars (show, don't just describe).
- **Context patterns (the catalog):** Externalize into memory (the write-bridge to Chapter 6), Just-in-Time Retrieval (pull via lightweight IDs, rank hard, keep the index fresh), Compaction (summarize before the window rots, watch context collapse), Context Isolation (sub-agents/sandboxes with clean windows), Cacheable Prefix (byte-stable, append-only, so the prefix caches), Recitation (restate the goal to fight drift), Keep the Errors (leave failures in so the model self-corrects; keep failures, guard false facts).
- *Aegis, patterned:* the incident agent assembled out of the named patterns.
- *In the wild:* LangChain's Write/Select/Compress/Isolate framework; Manus (Recitation, Keep the Errors, cache-hit-rate as the top metric); Claude Code and ADK (auto-compaction, caching via `static_instruction`).
- *When not to use these:* short, single-shot tasks with a small, stable context need almost none of them.
- *Which phase:* prompt and context (selection), with the runtime work handed to the harness.

### Chapter 6. Memory Patterns
- **Copy, paste, start over:** the pain memory ends. Context is *selection* (what's in the window); memory is *storage* (how to keep things so you can select the right one later). Memory serves context.
- **The four types (CoALA):** working (the window), episodic (what happened), semantic (facts), procedural (how-to). The movement between them, episodic to semantic to procedural, is how an agent gets wiser, not just fuller.
- **How memory fails:** amnesia, staleness, bloat, contradiction, leakage.
- **Where it lives (choose the store first; format decides what's possible):** Tiered Memory (core/recall/archival, MemGPT/Letta), Temporal Knowledge Graph (Zep/Graphiti, for relational and "as of when" queries), Scoping (namespace by user/session/agent/org).
- **The pipeline:** capture (Extract, Consolidate, Reconcile), recall (Score and Retrieve, blend relevance/recency/importance), maintenance (Reflect to synthesize, Prune to forget).
- **Two axes over the pipeline:** Control (programmatic vs agent-managed) and Timing (hot-path vs background). Reflect and Prune usually share one background "dream" pass, which is why they feel like one operation.
- *Aegis remembers:* the incident agent's four memory types and the capture-recall-maintain pipeline running on incident close.
- *In the wild:* Claude Code (markdown + grep + Dreaming, skips graph and scoring) vs Zep/Graphiti (temporal graph + hybrid scored retrieval); Mem0's extract-consolidate pipeline.
- *When not to use it:* stateless single-shot tasks.
- *Which phase:* context (what it knows) implemented as a harness responsibility.

### Chapter 7. Tooling & Tool Design Patterns
- **APIs as tools, not CRUD:** designing capabilities agents can reason about.
- **Function calling with typed schemas;** structured parameter descriptions.
- **MCP as the universal connector;** abstract tools over brittle endpoint wiring.
- **Tools for grounding:** LLM reasons about *what*; tools supply *accurate data* and side-effects.
- The tool-count ceiling (~20–50): when to split into sub-agents or low-level primitives.
- *Aegis:* runbook-execution tools and a metrics-query tool exposed over MCP, with typed schemas.
- *In the wild:* Spotify Honk's abstract verification tools over MCP as the glue to the build system.
- *Vendor lens:* AWS AgentCore Gateway centralizes MCP tools behind one endpoint; Google's tool-governance layer tackles duplicated per-agent tool-building; OpenAI frames the SDK as a "harness" that keeps tool execution close to the model's natural operating pattern.
- *Design Debate (code vs. visual):* code-first tool wiring (ADK, Strands, Agents SDK) vs. visual/registry approaches (AgentKit Connector Registry, Agent Studio, Foundry) — who is the "agent developer"?
- *When not to use it:* deterministic, static integrations where MCP is overkill.

### Chapter 8. Human-in-the-Loop Patterns
- **Framing:** "Nobody reads the diff." A gate that is always approved is not a control, it is a liability with a signature on it. Oversight is designed, not added, and the scarce resource is reviewer attention.
- **Reversibility, not importance, is the gating axis** (crossed with blast radius): autonomous+logged, autonomous+after-the-fact review, gated, always gated.
- **How oversight breaks:** rubber-stamping (theatre of approval), gating the trivia, blocking the loop (sync timeouts), stale approval, the silent override.
- **Deciding where the gate goes:** Tier by Reversibility; Put the Gate in the Orchestrator, Not the Model (a confident model skips gates exactly when novelty demanded one); Escalate on Uncertainty (confidence/value/anomaly thresholds).
- **Making the pause work:** Interrupt Asynchronously on Durable State (checkpoint, queue with TTL, resume; sync holds die to gateway/serverless timeouts and OAuth expiry); Re-validate on Resume; Timeout Policy with an explicit fail direction.
- **Making the human effective:** Write an Evidence Pack, Not an Alert (decide in under 15 seconds); Offer Four Verbs (approve, edit, reject, respond); Meet People Where They Work (Slack, the PR, the ticket).
- **Learning from the human:** Capture Every Override (audit trail + eval set + bug report); Earn Autonomy with Evidence (progressive autonomy, per action, not per agent).
- **The ladder:** in-the-loop to on-the-loop to out-of-the-loop; the review-throughput bottleneck forward-references Chapter 24.
- *Running example:* TODO (see front matter). Tiering, the async interrupt, the evidence pack in the on-call channel, an edited remediation captured as an eval case.
- *In the wild:* Honk landing work as reviewable PRs (the review surface predates agents); LangGraph static breakpoints + dynamic interrupts over a checkpoint store; enterprise maturity ladders moving deliberately from HITL toward human-on-the-loop.
- *When not to use it:* low-stakes reversible actions where a gate adds latency and nothing else; and "liability laundering," a gate that exists only to attach a human name to a failure.
- *Which phase:* the loop, with the harness supplying checkpoint/durable execution and policy supplying enforcement.

### Chapter 9. Evaluation Patterns
- **Framing:** "The green checkmark that lied." Of 58 agent traces with perfect outcome scores, ~83% still contained a procedural violation. Outcome hides how the answer was reached. Evaluation is an instrument you steer by, built first, not a launch gate.
- **The reframe (replaces "when not to"):** you always evaluate; the only question is *how much*, scaled by autonomy and cost-of-wrong.
- **Evaluation happens at many stages** (map before the techniques): the model itself (base benchmarks + your own fine-tune in isolation); development with synthetic data (build before real traffic); the evaluator itself (judge/reward model is a system under test); historical replay (play a new version against old production data; compare to incumbent); live production. The patterns are the techniques; the stages are where you apply them.
- **The patterns (flat catalog):** Score the Path Not Just the Destination (trajectory + outcome); Deterministic Checks First, Judges Only for What Resists Them; Judge but Validate the Judge (calibrate against humans; judge quality is itself measured); Prefer Comparison to Scoring (pairwise/arena beats absolute); Curate a Golden Set and Grow It from Failure (hand-built anchor, production regressions, overrides from Ch 8); Simulate Before You Let It Loose (sandbox, regulated-system treatment, earns the autonomy ladder); Run the Suite on Every Change (eval is CI for agents); Evaluate in Production on a Sample (cheap checks on 100%, judge on 5-10%, user signals, drift alerts, flywheel).
- *Running example:* TODO (see front matter). Simulation harness replaying historical incidents; deterministic checks + calibrated LLM judge; trajectory scoring that fails a right-answer-wrong-path run; overrides become golden cases.
- *In the wild:* evaluation as platform infrastructure (Google ADK eval module with trajectory tests on every code push; managed runtimes shipping prebuilt evaluators; open-source frameworks for judges + calibration + CI); the 2026 research turn to outcome/trajectory/meta and judge-as-measured-object.
- *Which phase:* wraps all four, but lives in the loop; the experiment that tells you whether every other pattern worked.

---

# Part III — Composition (Loop Engineering)

*This is the heart of the book. Part II built the pieces; here we compose them into working agents, then into systems of agents. The part opens on the multi-agent vision (the org chart), then earns its way there: get one agent's loop right first, make as much as possible deterministic, introduce controlled non-determinism, then coordinate agents, validate the loop, and finally let it improve itself. "Loop engineering" is used inside as the working term for the craft, but treated as a term-of-the-moment, not a durable label.*

### Chapter 10. Composing Agents
- **Framing (Part III opener):** "The org chart." The ambition is a fleet of agents that behave like a team (router, researcher, specialist, reviewer). But earn it: a fleet of unreliable agents is a committee that loses your data, so get one agent right first. Every box in the chart is a loop under the hood.
- **The heartbeat:** the loop is what turns Part II's parts into an agent. Reason, act, observe, repeat. Lineage: ReAct (2022) to Reflexion to modern coding-agent "while not done" loops.
- **A loop is not a chain:** a chain is linear and fixed; a loop decides the next step at runtime. Who chooses the next step is the fork the rest of Part III lives on. Most real systems blend a planned skeleton with adaptive loops.
- **The five parts of a well-built loop:** a testable goal, per-turn context (Ch 5), actions with honestly fed-back results incl. errors (Ch 7), a termination policy, an error strategy.
- **The loop shapes (noun handles):** the ReAct Loop (start here), the Plan-Then-Execute Loop (fewer calls, inspectable plan), the Reflection Loop (critic; seed of Ch 14), the Orchestrated Loop (multi-agent, Ch 12), the Human-Gated Loop (Ch 8 as structure).
- **Termination is the whole ballgame:** an LLM has no concept of "done." Anchor stat: ~60% of production LLM failures are rate-limit errors from runaway loops (Datadog). Layered exits: verified goal, iteration cap, no-progress detection. Build them on day one.
- *Running example:* TODO (see front matter). The top-level trigger loop; the five parts concrete; runaway-loop termination made vivid; names its shape (ReAct + human gate on remediation).
- *In the wild:* coding agents as long ReAct loops with hard exits (better-engineered loops around comparable models); LangGraph making control flow an explicit, designable graph.
- *Which phase:* the **Loop** (fourth phase from Ch 3), expanded from a list item into the discipline the rest of Part III unfolds; bookended by the org chart assembling itself as loops start handing work to each other.

### Chapter 11. Workflow Patterns and Shapes
*(Merged former Ch 11 Deterministic Workflows + Ch 12 Non-Deterministic Split, rewritten so the patterns are ARCHITECTURE shapes, how people actually wire deterministic + non-deterministic pieces, each carrying its own seam-check. Single metaphor: the wall of red you turn black, then wire the red that stays into safe shapes.)*
- **The wall of red (scene + thesis):** shade every unpredictable box; most first drafts are nearly all red, but most red is habit (a lookup, a date conversion, a total belong in code). The most agentic thing you can do is use less agent. The chapter: turn what red you can to black, and wire the rest into shapes that fence it.
- **Why you want black:** testable, auditable, cheap/fast, reliable. Default inverts: start all-deterministic, add red only where black can't do the job (the knowable-right-answer test).
- **The one rule every shape obeys:** model may interpret/classify/summarize/draft/rank/propose/route; may not approve/pay/provision/submit/delete/write-to-system-of-record without a deterministic check. Proposals vs consequences. Every shape places a **seam-check** where red hands to black, drawn from one menu: constrain the shape (constrained decoding), validate (semantic + retry-with-error), verify (independent/self-consistency), abstain (confidence threshold to a human), authorize (policy layer, Ch 16).
- **The architecture shapes (noun handles; each: shape, tradeoffs, where the seam-check goes, vendor instance), ordered by how much control the model holds, with the Channel Adapter appended last since it wraps rather than competes:** The Chain (fixed pipeline, prompt chaining); The Router (model labels, code dispatches to specialists); Parallel and Vote (sectioning for speed, voting/self-consistency for confidence, deterministic aggregation); The Orchestrator-Worker (model plans at runtime, deterministic harness executes; the reasoning-core shape; plan-then-execute); The Evaluator-Optimizer (generator + independent critic loop, deterministic stop); The Autonomous Loop (model owns control flow, fenced by Bounded Authority everywhere; hands to Ch 12); The Channel Adapter (many front doors, one canonical request; pure black; per-channel adapters authenticate, normalize, and stamp provenance/trust, renderers on the way out; channel content is data, never instructions; OpenClaw).
- **Choosing a shape:** start at the top, stop at the first that fits; nest them (Router to Chains, a Chain using an Evaluator-Optimizer); every step down trades predictability for flexibility.
- *Running example:* TODO (see front matter). Channel Adapter normalizes webhook/Slack/cron alerts into one canonical incident stamped with origin and trust; Router; per-category Chains, Orchestrator-Worker only if remediation is genuinely multi-step; mostly black with small red boxes at the seams, not an autonomous loop.
- *In the wild:* Google first (ADK workflow-agent primitives express the shapes; deterministic graph, red/black boxes), then Anthropic's five-workflow vocabulary (the reference names; simplest-shape-that-works; multi-agent last), Microsoft Workflows-vs-Agents, AWS AgentCore Policy (authorize seam as infra).
- *When not to over-shape:* keep high-assurance paths fully black; don't impose a heavy shape when a single call or 3-step Chain does the job.
- *Which phase:* the **loop** at its seam; every shape is a way of arranging that boundary.

### Chapter 12. Multi-Agent Orchestration
- **Framing (reframe):** "The org chart lies." Ch 10's org chart is the right picture and the wrong architecture: a real org runs on people talking to each other, and agents that talk to each other fragment context. **Major update vs the original outline: the multi-agent debate has substantially resolved.** Cognition published "Don't Build Multi-Agents" (2025) then shipped an orchestrator managing its own instances (2026); Anthropic shipped role-scoped subagents; OpenAI made nested handoff history opt-in; AutoGen folded into Microsoft's framework with peer GroupChat demoted; LangChain moved to supervisor-as-tool. Five independent teams converged on orchestrator-with-isolated-subagents. Peer-to-peer lost.
- **The thing that actually breaks:** context inconsistency. Cost, latency, and debuggability are symptoms. The anchor example: a supervisor fans a financial query to Finance and Compliance specialists and gets two confident answers built on different definitions of "revenue," held in separate memory stores, with no principled way to resolve them.
- **The rule:** context has one owner. **Durable part:** one owner. **Dated part (2026 trend):** that owner is a single orchestrator, stated as a trend rather than a law. **Why it won:** it hands you a **consolidator** for free (something must merge returns and decide what is next, easiest when one thing does it). That makes it a convenience, so hold it loosely: the Pipeline has no orchestrator and an event-driven shared-log design has no central agent, and both satisfy the rule. What none of them do is leave the question of who consolidates unanswered. One agent owns the full context; delegated work goes to a subagent whose context is **seeded from above and owned locally** (it is a real agent, so it runs its own loop and accumulates its own history, including back-and-forth with the orchestrator), returns a compressed summary, and has its local copy discarded. **Vertical traffic (parent-child) is fine, because the orchestrator owns the truth at both ends; horizontal traffic (sibling-sibling) is forbidden.** But state the invariant narrowly so it ages: **the enemy is divergent copies, not shared access.** One authoritative picture with clear write-ownership is what matters; an orchestrator is the simplest way to get it, and a shared append-only log every agent reads (event-driven designs) satisfies the rule too.
- **Agents, tools, and who holds the state (before the topologies):** the tool/agent line is **state**, not size. A tool is stateless (args in, result out, remembers nothing; the agent holds all persistence). An agent is stateful (accumulating context, memory, loop). Promoting a tool to an agent creates a second place state lives, which is where context inconsistency starts. **Three rungs, climb only when forced:** a **tool** (stateless call); an **agent as a tool** (stateless call with a loop inside; a one-off that reasons internally, returns, and forgets the flow; lives in the tool list, not the topology; LangChain's supervisor-as-a-tool); a **delegated agent** (the only rung that is a task rather than a call: lifecycle, vertical conversation with the parent, context carried across the flow). Key line: statelessness is about what survives the call, not what happens inside it. The protocols split on the same line (MCP = call, A2A = task). Default: **prefer a tool.** When you do delegate: **one copy of the state, with the full history.** Each agent keeps its own conversation as it works (that is what a loop is, and it is fine); what matters is that one place holds the canonical record of what is known, what was delegated, what came back, and in what order, with writes scoped to keys an agent owns. The failure is a private long-lived copy of a shared fact edited on the side. This is how the frameworks actually work: an **ADK Session is that canonical copy** (ordered event history + `session.state` working facts; `output_key` writes, prefix scoping, `{key}` templating), while each agent still runs its own conversation; LangGraph is typed state + reducers + checkpointer. Both say persist it outside the runtime for production, which is what makes replay real. Payoff: state you own is state you can **replay** (re-run, resume after a crash, reconstruct a failure); state spread across siblings is unreplayable. The protocols encode the distinction: MCP is agent-to-tools (unit = a call that returns), A2A is agent-to-agent (unit = a task with a lifecycle, because the far end is stateful). A2A under the Linux Foundation, native across the major clouds; the unlock is cross-vendor. Build on the durable primitives (identity, task, message, status, artifact). *(Ch 7's Tool Placement pattern now names statelessness explicitly to set this up.)*
- **When you actually need more than one (only three reasons):** the context will not fit; a boundary must be enforced (security/compliance isolation); the work is genuinely parallel and independent. Anything else, build one agent and fix it.
- **The topologies (noun handles, ordered by how much the agents talk = how much context can fragment; each: shape, tradeoffs, where the context lives, in the wild):** The Pipeline (one-way handoff, context lives in the payload); The Supervisor (the 2026 default, orchestrator owns context, ephemeral isolated subagents, one level deep, summary quality is the whole ballgame); Fan-Out (parallelism only, disjoint slices, deterministic merge); Debate and Judge (deliberate redundancy, ~2.5x cost, the one place duplicated context is a feature); Peer-to-Peer (the shape that lost: O(n^2) relationships, full-transcript re-reads, no authoritative picture; swarm as its frontier extreme).
- **What the shapes have in common (after the topologies):** reconciles the shapes with the state rule. Pipeline has no top (state travels sideways); Debate duplicates on purpose. Neither breaks the invariant once stated generally: each shape is a different **write-scoping discipline over one shared store**. Supervisor (orchestrator owns the schema; subagents get scoped views + a result key); Pipeline (writes owned by position in the chain); Fan-Out (disjoint keys, which is what "independent" operationally means); Debate (each writes its own key, one judge writes the verdict, so duplication is safe); Peer-to-Peer (no key ownership, drift cannot resolve). **Invariant: every fact in the shared store has exactly one writer, known before the run starts.** Reads can be free.
- *Running example:* TODO (see front matter). Does it need more than one agent? Walk the three reasons; land on a Supervisor with a log-analysis subagent and Fan-Out over service queries; explicitly do NOT let diagnosis and remediation agents talk.
- *In the wild:* Google first (ADK treats multi-agent as the default deployment mode, A2A-native, supervisor-to-workers reference architecture that is cross-vendor by default), then the convergence story across Anthropic, Cognition, OpenAI, Microsoft. "When the loudest critic and the most committed vendor build the same shape, that shape is the answer."
- *When not to:* one agent is the default and stays it longer than you expect. If you cannot name which of the three reasons applies, you do not have a reason.
- *Which phase:* the **loop** at scale; many loops and the question of who owns the truth across them. Comes after the split because you cannot orchestrate agents you have not made reliable.
- *Design Debate sidebar (still to write):* "the argument and how it ended," with the Cognition reversal as its spine.

### Chapter 13. Engineering the Loop
*(Rewritten per author: this chapter is about **designing the loop at each scale** and how the Ch 11 shapes and Ch 12 topologies nest and combine. An earlier guardrails-catalog draft was rejected; Ch 10 already carries termination basics.)*
- **Framing:** "Where is the loop?" The diagrams so far say what the pieces are; the loop says what actually runs. Once you have more than one agent you do not have a loop, you have several running inside each other.
- **The loop in one agent:** one loop, one context, one budget, one exit, one trace. Nobody asks who is in charge, because there is only one candidate. **The tradeoff:** cheapest and most debuggable thing in the book; capped by one window and one toolset, no parallelism, no specialization. The failure is specific: the context fills and quality falls off a cliff that has nothing to do with the model.
- **The loop with sub-agents (the load-bearing idea):** **an inner loop collapses into one step in the outer loop.** The parent calls, waits, gets a result; delegating looks exactly like calling a slow tool. **The Supervisor topology lives here**, not with the many-agent shapes, because it has a clear orchestrator (frameworks agree: Anthropic's agent SDK caps subagents at one level deep). Three things nest: **budgets** (and multiply: 10 x 10 = 100 calls; allocate downward as a share), **termination** (an inner cap produces a failed step the parent reads as an observation, not a dead run), **traces** (a tree; collapsible, but the summary is where the information you need got thrown away). **The tradeoff:** buys context isolation and specialization; pays multiplied tokens, added latency, and a lossy summary boundary (a bad summary makes the parent confidently wrong). Two costs that surprise people: the orchestrator accumulates context from every worker and at 4+ frequently blows its own window (the problem you delegated to avoid), and goal drift from a vague original request.
- **The loop across many agents, what ends the loop?** Take away the orchestrator and nothing decides unless you build it. **Fan-Out ends when something consolidates**: the join is designed last, and you need a reducer that handles a hung branch and two branches disagreeing; sometimes the reducer must itself be an agent. Slowest branch sets the pace. **Debate ends when the judge says so**, with bounded rounds behind it; without the judge it is oscillation with better manners. **Peer-to-Peer does not end**: each agent has an exit, the system has none, and the thing that stops it is your rate limit. **The tradeoff:** buys independence and real parallelism; pays by hand-building the exit, the consolidation, and the accountability the orchestrator gave you free. "Ask what ends the loop before you draw the second box."
- **Combining the shapes:** any shape can be a step inside any other shape, which is the whole grammar. Chain with an Autonomous Loop as step 3 (most common real architecture, reach for it first); Supervisor whose workers are Chains; Evaluator-Optimizer wrapping a Fan-Out; a Router in front of everything (which does not loop). The question to ask at every level: what ends this loop, and who consolidates what comes back?
- *Running example:* TODO (see front matter). Walk it up the scales: single loop, then a log-analysis sub-agent (20 turns in, two sentences out, one step to the parent; 10 x 20 = 200 calls at 3 a.m.), then the composition it settles into (Router to per-category Chain to bounded Autonomous Loop to Fan-Out).
- *In the wild:* Google first (ADK composes by containment, `SequentialAgent` holds `LoopAgent` holds `LlmAgent`; its examples ship Chain-with-an-Autonomous-Loop as a constructor arg; workflow agents deterministic so the driving loop is readable in the structure), then Anthropic's SDK (supervisor native, subagents one level deep), OpenAI handoffs, LangGraph graph-as-node; then the many-agent evidence: a published multi-agent financial-document study needing a **dedicated reconciliation agent** to merge conflicting fan-out branches, and durable-execution engines checking the done flag **and** an iteration cap with compensation after real-world actions.
- *When not to:* nest only when forced; ask of every sub-agent whether a tool would have done it, because usually one would.
- *Which phase:* the **loop** as an architecture rather than a mechanism.

### Chapter 14. Self-Evolving Agents
*(Per author: combines **the loop with memory** (the dream agents) plus recent self-evolving engineering-agent work. Author's sharpened brief: after consolidation, distillation, and generalisation, **the patterns are the ways to evolve an agent**, so the catalog is the *targets* of evolution, not the mechanisms.)*
- **Framing:** "What happens between sessions." The loop ends, the context is discarded, everything it worked out dies with it. "Your agent is not learning. It is auditioning, every single day, for a job it has already done a thousand times."
- **Consolidate, distil, generalise (the setup):** the loop produces **episodes**, mostly noise. **Background Reflection** (*read many episodes during idle time and write down the few things that generalize*) runs off the hot path, because consolidation is slow, costs tokens, and helps nothing in flight, "which is why everyone who has built this reached for the same metaphor and named it after sleep." This is Ch 6's **Reflect** on Ch 6's background timing axis; Google's "reflective memory" is the same thing. Deflation retained: storing and retrieving is retrieval; evolution is converting experience into persistent behavioural change. "Plenty of systems retain. Far fewer convert."
- **Six things you can change (the ladder):** consolidation hands you a lesson; the question nobody answers is where to put it. Six rungs, and climbing costs you: **blast radius** grows (a note affects retrievals, a prompt affects every run), **reversibility** falls (delete a line vs retrain), **feedback slows** (tonight vs three weeks), **auditability** drops (read a runbook vs read a weight), **cost** rises. **The discipline: put the lesson on the lowest rung that can hold it.** "Most teams reach for the top rung first, because it is the one that sounds like machine learning."
- **The rungs (noun handles, 2-3 sentences each):** **Updated Notes** (a fact; cheapest, instant, one keystroke to delete); **New Skills** (a routine; one distilled skill stands in for twenty trajectories); **Rewritten Instructions** (always-true things folded into the prompt; watch the blast radius, the prompt is a shared resource that fills up); **New Tools** (a procedure re-derived every time is a tool waiting to happen; **moves it from red to black**, testable and free, but it is agent-written code so it needs a sandbox and a review); **Changed Code** (router categories, thresholds, workflow branches; highest leverage, changes the parts that were reliable to begin with, so it should arrive as a **pull request a human merges**, not something the agent does at 3 a.m.); **A Tuned Model** (distil trajectories into training data and move the weights; for when a behaviour must be instinct rather than instruction, trading auditability for a capability the other rungs cannot reach).
- **The filter is the whole job (the top rung's warning):** training on your own trajectories means training on your own mistakes unless you filter. Standard answer keeps only successful runs, which works and is brutally wasteful (**one well-known agent dataset throws away ~60% of collected runs**). But **only ~24% of steps in failed trajectories are actually mistakes**, so the rest was good work discarded for having a bad ending; filter **step by step, with a critic**, not run by run. "An unfiltered dataset is worse than a smaller filtered one."
- **The gate (applies at every rung):** **no learned change reaches production without passing evaluation** (Ch 9's golden set; if scores regress it does not ship). Two rules travel with it: **keep the learning readable** (you can argue with a runbook, which is why the bottom rungs are safer than the top), and **bound what the agent may change about itself** (notes and skills yes; its guardrails, policy, and termination conditions never). "The distinction is between an agent that gets better at its job and an agent that renegotiates the job."
- **What goes wrong:** a bad lesson persists (one unlucky run becomes permanent, retrieval spreads it); the library rots (consolidation adds, almost nobody builds the thing that removes); drift; reward hacking (whatever you count, it optimizes, and it will not tell you it found a shortcut).
- *Running example:* TODO (see front matter). Walk it up the ladder: Background Reflection spots the same failure four times, then place the lesson (Updated Note for the ownership fact, New Skill for the runbook, New Tool once it has hand-rolled the same queries fifteen times, Changed Code as a PR for the mis-categorising router). Say why it never reaches the top rung: volume too low, feedback too slow, and the on-call needs to read why it did what it did.
- *In the wild (walks the ladder):* Google first (**"Language models need sleep"**, consolidation as first-class architecture; Antigravity reflective memory; Vertex Memory Bank; the framing to steal: **idle time is a resource you have already paid for**). Then **Anthropic Dreaming** (May 2026) as the productized bottom two rungs, and what makes it interesting is **what it refuses to do**: no weights, everything human-readable and rejectable, ceiling honestly stated. "They took the lower ceiling to keep the audit trail." Then the coding agents at the top rung (tests pass or they do not; the ~60% discard; the ~24%-of-steps finding), with the bottom rungs still running underneath (Claude Code mid-session memory writes; skill libraries at ~10-20x compression). **"Nobody serious is on one rung."**
- *When not to:* stability and auditability beat adaptation in regulated domains; and when the work is genuinely novel every time there is nothing to consolidate.
- *Which phase:* the **loop**, closed. The end feeds the beginning through memory.

# Part IV — Hosting Agents

*You've built the agent (Part II) and composed its behavior into working loops (Part III). Now run it for real. This part is the operational substrate: where an agent is hosted, how it's isolated, how it proves who it is and finds what it's allowed to use, how you watch what it does, and the policy that governs what it can do. It's the production layer the case studies in Part V lean on, and the difference between a demo and something you'd deploy.*

### Chapter 15. Sandboxing and Runtime
- **Where agents run:** managed agent runtimes (Vertex Agent Engine, Bedrock AgentCore Runtime, Azure Foundry) vs. containers vs. serverless vs. self-hosted; the own-vs-govern-the-loop ladder (Ch 2) applied to hosting.
- **Sandboxing and isolation:** running agent-generated code and risky tool calls in a sandbox (microVMs, gVisor, ephemeral containers, hosted code sandboxes); session isolation so one run cannot see another; containing the blast radius. Cashes out Ch 11's Code Execution rung and Ch 14's New Tools rung, both of which hand the agent the ability to write and run code.
- **Durable execution and state:** checkpointing, resumable long-running agents, state that survives a crash. Cashes out Ch 8's async HITL pause and Ch 12's replayable record.
- **Scaling the runtime:** horizontal scale, mostly-stateless workers, concurrency, and the cost of keeping them warm.
- *Running example:* TODO (see front matter). Each remediation in a fresh sandbox; managed runtime with durable state so a long incident survives a restart.
- *In the wild:* Google first, then microVM isolation, hosted code-execution sandboxes, durable-execution engines.
- *When not to use it:* a read-only agent that never executes code or writes state needs little of this.
- *Which phase:* harness.

### Chapter 16. Identity, Discovery, and Policy
*(Merged per author from the former Ch 15 "Identity and Discovery" and Ch 18 "Policy and Guardrails". They are one concern: identity is the prerequisite for policy, since you cannot authorize an agent that cannot prove who it is, and discovery is itself an authority question, because what an agent can find should be a function of what it may use.)*
- **Verifiable agent identity:** who the agent authenticates as; SPIFFE/mTLS between nodes; A2A Signed Agent Cards; agents as first-class principals rather than static tokens borrowed from a human.
- **Runtime capability discovery:** agents query a registry for approved tools and APIs instead of hard-coding endpoints (MCP registries); machine-readable capability specs so an agent can reason about what a capability does before calling it. Discovery is scoped by identity: an agent should only be able to find what it is allowed to use.
- **Just-in-time authorization:** the agent decides at runtime what to call, so permissions are scoped dynamically rather than provisioned up front.
- **Guardrails in IAM and policy, not the prompt:** "don't query the PII columns" belongs in access policy, where the model cannot talk its way past it. This is where Ch 11's Bounded Authority and Ch 7's Guardrailed Boundary actually live.
- **Content safety and injection defense:** input/output filtering, RAI middleware, indirect-prompt-injection defenses (Model Armor). Ch 11's Channel Adapter stamps provenance; this is the layer that acts on it.
- **Blast-radius containment for chained execution:** preventing privilege escalation across autonomous tool chains; high-stakes actions routed to a human gate (Ch 8).
- **Governance and audit:** every action attributable, logged, reviewable.
- *Running example:* TODO (see front matter). Discovers only the remediation tools it is authorized for; authenticates as itself so every action is attributable; "never touch the customer DB" enforced in IAM; a human gate before any production write.
- *In the wild:* Google first (Model Armor, zero-trust, agent identity in the platform), then AWS AgentCore Identity/Policy, Microsoft RAI middleware, MCP registries, A2A identity, the 2026 push toward verifiable agent credentials.
- *Design Debate (guardrail placement):* IAM/policy (AWS, Google) vs. middleware pipeline (Microsoft) vs. model/harness (Anthropic, OpenAI), the same guardrail, three homes.
- *When not to use it:* a single closed agent with a fixed toolset needs no discovery layer; read-only low-stakes agents do not need the heavy policy machinery (say so explicitly).
- *Which phase:* harness, plus the social contract.

### Chapter 17. Observability for Agents
- Beyond p95s and error budgets: **"what did the agent decide, and why?"**
- Capturing prompts, tool calls, intermediate decisions, and final outputs.
- Agentic metadata as an infrastructure layer; tracing across a multi-agent run (Ch 13's trace tree, where an inner loop collapses to one node and the summary is where the information you need got thrown away).
- Turning traces into eval data (Ch 9) and into the improvement signal for self-evolving loops (Ch 14). **Observability comes last in this part because it is what feeds the loop back to the beginning.**
- *Running example:* TODO (see front matter). A decision trace for every incident, feeding both the eval set and the runbook-promotion pipeline.
- *In the wild:* Google first, then AWS's framing that traditional APM misses the bug that matters most, a *valid-but-wrong decision* in a multi-step workflow, so instrument the reasoning trace from the start; OpenTelemetry as the shared standard across Microsoft Agent Framework, AgentCore, and third-party tools.
- *Which phase:* harness.

---

# Part V — The Landscape and Case Studies

*A survey chapter, three scenario-driven design studies (each anchored on a different dominant force), and two real teardowns. The design studies are fresh systems, not Aegis, chosen to show the same patterns behaving differently when scale, the edge, or a merger is the thing that reorders every decision.*

### Chapter 18. The Vendor Landscape — Convergence and Its Discontents
- **The thesis:** the five major platforms have converged on nearly the same reference architecture (reasoning model + tools + memory + orchestration + identity + observability) and the same two protocols (MCP for tools, A2A for agents) — proof that Parts I–IV describe an industry consensus, not one opinion.
- **The convergent core** (the vendor-neutral reference model): deterministic/probabilistic split; MCP + A2A; memory as a first-class layer; identity & security as foundations; built-in decision tracing; model-agnostic managed runtimes; simplicity-first.
- **Five instantiations, one page each** — how each platform expresses the same patterns:
  - **AWS** — Bedrock AgentCore (Runtime/Gateway/Memory/Identity/Observability/Policy/Evaluations); ops-first; "is this really an agent?" gate; the *valid-but-wrong decision* as the bug that matters.
  - **Anthropic** — *Building Effective Agents* (simplicity, transparency, agent-computer interface); context as a finite attention budget; reduce abstraction in production.
  - **Microsoft** — Agent Framework 1.0; the Agents-vs-Workflows separation as a foundational decision; five orchestration patterns; RAI middleware pipeline.
  - **Google** — Gemini Enterprise Agent Platform (ADK / Agent Studio / Agent Engine); A2A authorship; Model Armor + zero-trust; own-the-stack bet; deterministic rules + probabilistic reasoning.
  - **OpenAI** — Responses API → Agents SDK → AgentKit; the thin, model-hugging harness.
- **The independent frame:** Andrew Ng's four patterns (reflection, tool use, planning, multi-agent); LangGraph's graph model; Cognition's "don't build multi-agent" caution; the academic pattern catalogues — used to keep the survey above any vendor's marketing.
- **The five debates** (forward-referencing the Design Debate sidebars): multi-agent, abstraction, guardrail placement, own-stack vs. agnostic, code vs. visual.
- *Why this chapter is here, not in Part I:* the reader needs the patterns (Parts II–IV) before the vendor mapping means anything; placing it as the case-study opener turns "here's the landscape" into the natural runway for the studies that follow.

### Chapter 19. The Peak-Day Marketplace — Designing for Scale
- The dominant force: volume, throughput, cost, and latency at millions of requests. When scale is the constraint, it reorders every decision.
- **The use case:** the shopping assistant on a giant online marketplace, the kind Amazon or Alibaba run. A few questions per second on an ordinary Tuesday, then 50x that on Black Friday or Singles' Day, when every extra 100ms and every wasted cent per query is multiplied across tens of millions of concurrent shoppers.
- Design moves driven by scale: multi-model routing and cascades (a cheap model for the 90%, a stronger one only for the hard 10%, Chapter 4); aggressive Cacheable Prefix and batch APIs (Chapters 4, 5); deterministic fast paths for common cases (Chapters 11, 12); async and background work; horizontal scaling of a mostly-stateless harness with durable state only where needed; sampled observability.
- The trade-offs: freshness vs. cost, per-request routing overhead, where to spend the frontier budget.
- *In the wild:* **Klarna's** OpenAI-powered assistant is the flagship scale case: ~2.3M conversations in month one, roughly two-thirds of support chats, framed as the work of ~700 (later ~853) agents, across 23 markets and 35+ languages, resolution 11 min to under 2 min, cost-per-transaction $0.32 to $0.19; grounded in help-center content with low-confidence cases routed to humans. Use the **2025 walk-back** as the cautionary half: humans rehired for complex, emotional, and dispute cases, and weak escalation handoffs (customers re-explaining themselves) as the failure that hurt CSAT. Secondary: Rocket AI Agent on Amazon Bedrock (3x conversion, 85% fewer transfers).
- Patterns that dominate: multi-model, caching, deterministic workflows.

### Chapter 20. The Delivery Courier — Designing for the Edge
- The dominant force: intermittent or absent connectivity, latency, and constrained hardware (and privacy, for some edge cases). Speed and locality win.
- **The use case:** the courier's assistant inside a food-delivery app, the kind Uber Eats or DoorDash run. It lives on the driver's phone, guiding the route and handling order changes and customer messages, and it has to keep working when they drop into a parking garage, a tunnel, or a rural dead zone, so it can't assume the cloud is there.
- Design moves driven by the edge: small or fine-tuned open models on-device (Chapter 4); local tiered memory with sync-on-reconnect (Chapter 6); graceful degradation and a local fallback model; a hybrid split, a small local model that escalates to a cloud frontier model only when connected; actions that queue locally and reconcile when the signal returns.
- The trade-offs: capability vs. footprint, staleness of local memory, the sync-and-merge problem.
- *In the wild:* **DoorDash, Uber, and Grubhub** testing multi-step agentic ordering in the Google Gemini app (Mar 2026), where Gemini acts in-app inside an on-device "secure virtual window" (edge sandboxing). The dispatch brains are already edge-shaped: Uber's Trip State Model and DoorDash's DeepRed engine run on real-time GPS and motion-sensor data, and last-mile driver apps are built around offline sync and geofencing. The hybrid split is a live research line: on-device inference on NPU-class phones (small Gemma/Phi/Qwen variants) with edge-SLM-to-cloud-LLM escalation.
- Patterns that dominate: multi-model (edge/small), local tiered memory, fallbacks.

### Chapter 21. The Bank and the Neobank — Designing for a Merger
- The dominant force: heterogeneity, governance, and identity across two organizations, a large enterprise absorbing a small one, each with its own stack, data, and risk culture.
- **The use case:** a century-old retail bank acquires a fast-growing neobank. The bank's mainframe-era core and the startup's cloud-native agents have to serve the same customers from day one, across two identity systems, two risk cultures, and two data regimes, without a two-year migration first.
- Design moves driven by the merger: agents as the glue bridging legacy enterprise and modern startup systems via MCP connectors (Chapter 7); identity federation and memory Scoping across two orgs (Chapters 6, 20); guardrails reconciling two risk cultures (the "social contract" from Chapter 2); data residency and access control; incremental migration with agents carrying the seam through a multi-year integration.
- The trade-offs: autonomy vs. governance, whose conventions win, how much to unify vs. bridge.
- *In the wild:* **TELUS Digital's** agentic-banking write-up is this scenario almost verbatim: a backend-for-frontend (BFF) plus MCP orchestration layer gives agents permissioned access to a 40-year-old mainframe without replacing it ("build above it"). **Strata's Maverics** does M&A identity integration via an abstraction layer over two orgs' identity providers, translating protocols instead of migrating identities. **Deloitte** describes agents that reconcile records and catalogs across legacy systems and flag conflicts, avoiding an immediate data migration. War story: a post-merger ERP consolidation (Western Digital, 25 legacy systems retired) using scout agents to map shadow processes from transaction logs. Strong enough that Ch 21 could run as a light teardown, not a pure hypothetical.
- Patterns that dominate: tooling/MCP, scoping, identity and guardrails.

### Chapter 22. Spotify Honk — A Coding Agent, Torn Down
- A background coding agent powered by Claude, built on the existing Fleet Management harness.
- Slack-native trigger; MCP-exposed verification tools; feedback loops for predictable results.
- Context engineering across thousands of repos; loop engineering as the reliability source.
- Outcome-based accountability; PRs as the human-review gate.
- Result: 1,500+ merged PRs; the shift from code *creation* to full-lifecycle work.

### Chapter 23. ServiceNow — The Enterprise Workflow Agent
- The scenario: agentic IT and enterprise-workflow automation (incident, request, and change management) across a large enterprise's connected systems.
- Architecture: LLM reasoning wrapped around deterministic, governed workflows; connected/specialized agents coordinating over enterprise data; approvals and audit built in.
- Why it's the counterweight to Honk: heavily governed, deterministic-first, and enterprise-integration-heavy, where Honk is developer-autonomous and loop-first.
- Lessons: deterministic workflow plus probabilistic reasoning; governance as a first-class design input; the enterprise-integration reality most production agents actually live in.
- *(Research current ServiceNow AI Agents / Now Assist specifics before drafting.)*

---

# Part VI — Frontier

### Chapter 24. The New Bottlenecks
- What breaks when agents ship work faster than humans can review it.
- Accountability, trust collapse, and the review-throughput problem.
- Agent "factories": coordinated agents producing complex knowledge work.

### Chapter 25. The Full Lifecycle
- Beyond creation: maintenance, migration, deletion — the work nobody wants.
- Agent-first platforms (developer portals becoming agent interfaces via MCP).
- Edge and alternative-cloud inference: proximity over raw processing power.

### Chapter 26. Identity, Trust, and the Agentic Web
- Machine identity as a top-tier security concern.
- Verifiable credentials, signed agent cards, and the non-human internet.
- Open standards (MCP, A2A) and where the ecosystem is heading.

---

# Appendices

- **A. The chapter-pattern quick reference** — every pattern on one page.
- **B. The Aegis codebase** — the running example, assembled end to end.
- **C. Architecture review checklist** — a printable one-page review checklist distilled from Parts II–IV. *(Was the Chapter 16 checklist; that chapter is cut, so this appendix now has to carry it or be dropped.)*
- **D. Framework & protocol cheat sheet** — ADK, LangGraph, CrewAI, Bedrock, MCP, A2A, Arazzo.
- **E. Glossary** — agentic misalignment, JIT authz, agentic metadata, plan-do-evaluate, and more.

---

## Notes on sequencing (for the author)

- **Parts I–IV are the backbone of the argument; Part V is proof; Part VI is the horizon.** A reader could stop after Chapter 16 for the complete design method, or after Part IV (Chapter 16) for the production-ready one.
- **The running example (TODO, see front matter) must be introduced in Chapter 1 and touched in every pattern chapter** or the "cumulative" feel is lost, this is the highest-risk thread to maintain.
- **Chapter 3 (the four phases) and the loop-engineering cluster (Chapters 10–11) are what sell the book.** If a reviewer reads only those, they should get the whole thesis.
- **Part III's internal order (Composition) is deliberate:** deterministic first (Ch 11), then the split (Ch 11), so the reader anchors on predictability before non-determinism. Loop validation (Ch 13) follows multi-agent (Ch 12) because validating a single loop is a prerequisite to trusting many. Self-evolution (Ch 14) closes the part by planting a need Part IV answers: you can't safely evolve what you can't observe, so observability (Ch 17) now lands as the operational concern it is, in Hosting Agents.
- **Part V mixes scenario design studies and teardowns.** Chapters 15–21 are fresh, Aegis-free system designs, each anchored on one dominant force (scale, edge, merger) to show the same patterns behaving differently under different pressure. Chapters 19–23 are real teardowns (Honk, ServiceNow) chosen as opposites: developer-autonomous and loop-first (Honk) vs. governed and deterministic-first (ServiceNow). Spotify Ads AI was cut; Shopify and Block survive only as in-text vendor-lens examples in Part II. The security split and the new design studies grew the book to 29 chapters, so watch the page budget.
- **The Design Debate sidebars are the neutrality mechanism.** Five recurring sidebars (mapped in "About this book") stage the real vendor disagreements at the chapters where they bite, presenting both camps rather than an answer. Given the author's Google affiliation, holding these scrupulously even-handed is what protects the book's vendor-neutral credibility — Google appears as one instantiation among five, never the default.
- **The vendor landscape (Ch 18) is one chapter, not a part — deliberately.** Instead of a chapter per platform (which bloats the page count and dates fastest), the convergence table is the single reference model and each vendor is a one-page instantiation. This is the main lever for hitting the ~500-page target while staying current.
- Keep case-study *and vendor* specifics (versions, metrics, product names) dated and quarantined in Part V and the appendices — branding shifts fast (e.g., Vertex AI Agent Builder → Gemini Enterprise Agent Platform within a single year), so the pattern chapters in Parts I–IV must stay product-free to age well.
- **Voice consistency is a first-class risk, like the Aegis thread.** Draft the *voiced* sections (openers, sidebars, closes) against the Voice & Style Guide, and resist letting the voice creep into the mechanics. If a chapter opener doesn't have a real scar or scene behind it, write it plainer rather than manufacturing an anecdote — a forced war story reads worse than none.

### Sources underpinning the examples
- InfoWorld, *Best practices for building agentic systems* (Apr 2026).
- Spotify Engineering: *Spotify x Anthropic Live* + Honk series (Nov 2025–Apr 2026) for the Chapter 22 teardown. ServiceNow AI Agents / Now Assist for Chapter 23 (research current specifics before drafting).
- IBM Think 2026 recap; A2A protocol v1.0; 2026 MCP roadmap.
- Vendor landscape (Ch 18): AWS Bedrock AgentCore docs & AWS ML blog; Anthropic *Building Effective Agents* (Dec 2024) and *Effective Context Engineering for AI Agents* (Sept 2025); Microsoft *Agent Framework 1.0* (GA Apr 2026) & Azure Architecture Center; Google Cloud Next 2026 / Gemini Enterprise Agent Platform docs; OpenAI *AgentKit* & Agents SDK; Andrew Ng (AI Ascent 2024); Cognition "Don't Build Multi-Agents"; Liu et al., *Agent Design Pattern Catalogue* (JSS 2025). *(Full detail in the companion vendor-landscape briefing.)*

### Design-study case sources (Ch 15–21) — for the author
*Recent online cases gathered to anchor the three design studies. Volatile: verify figures and check for newer coverage before drafting.*

- **Ch 19 (Scale) — Klarna AI assistant:**
  - Klarna press release, *AI assistant handles two-thirds of customer service chats in its first month* — klarna.com/international/press/klarna-ai-assistant-handles-two-thirds-of-customer-service-chats-in-its-first-month/
  - LangChain case study (built on LangGraph/LangSmith), Jun 2026 — langchain.com/blog/customers-klarna
  - CX Dive, cost-per-transaction $0.32→$0.19, May 2025 — customerexperiencedive.com/news/klarna-ai-slash-customer-service-costs/748647/
  - Twig, deep analysis incl. the 2025 walk-back — twig.so/blog/klarna-ai-customer-support-efficiency
  - ZenML LLMOps DB (architecture summary; also Rocket AI Agent on Bedrock) — zenml.io/llmops-database/ai-assistant-for-global-customer-service-automation

- **Ch 20 (Edge) — food-delivery / on-device:**
  - Stellagent recap of Retail TouchPoints: DoorDash/Uber/Grubhub agentic ordering in Gemini; on-device "secure virtual window", Mar 2026 — stellagent.ai/insights/doordash-uber-gemini-agentic-ordering
  - Riseup Labs: Uber Trip State Model, DoorDash DeepRed dispatch, DashAI, Jan 2026 — riseuplabs.com/ai-in-food-delivery-app-development/
  - Bolder Apps: on-device inference, NPUs, AI-native mobile apps, 2026 — bolderapps.com/blog-posts/beyond-the-chatbot-5-ways-ai-native-mobile-apps-are-disrupting-traditional-service-industries-in-2026
  - arXiv survey: *Collaborative Inference and Learning between Edge SLMs and Cloud LLMs* (2507.16731) — the edge/cloud hybrid split

- **Ch 21 (Merger) — M&A integration:**
  - TELUS Digital: *Agentic banking without core replacement* (BFF + MCP bridge over the mainframe), May 2026 — telusdigital.com/insights/data-and-ai/article/agentic-banking-without-core-replacement
  - Strata.io: Maverics identity orchestration for M&A (abstraction layer over IDPs), Apr 2026 — strata.io/blog/app-identity-modernization/managing-identity-in-a-merger-acquisition-or-divestiture/
  - Deloitte: *Where is the value of AI in M&A* (multi-agent + modern data architecture; reconciliation agents) — deloitte.com/cz-sk/en/services/consulting/blogs/where-is-the-value-of-AI-in-MA-why-multi-agent-systems-needs-modern-data-architecture.html
  - CIO / Puneet Thakkar: *From go-live to always-live* (scout/user-proxy agents; Western Digital ERP consolidation), Feb 2026 — cio.com/article/4128116/from-go-live-to-always-live-how-agentic-ai-is-rewriting-the-finance-transformation-playbook.html
  - Accenture: *Agentic AI reshaping M&A*, May 2026 — accenture.com/us-en/insights/strategy/agentic-ai-reshaping-mergers-and-acquisitions
  - Forbes: *How AI speeds up post-merger IT integration*, May 2026 — forbes.com/sites/cio/2026/05/21/how-ai-speeds-up-post-merger-it-integration/
