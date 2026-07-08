# Agentic System Design
### Patterns for Engineers and Architects Building Autonomous Systems

*Full chapter-by-chapter outline / table of contents (working draft)*

---

## About this book

**Audience:** engineers and architects designing, building, and operating agentic systems — people who will write the harness, wire the tools, and get paged when the agent does something surprising.

**Promise:** by the end, the reader can look at any agent requirement and decompose it into the right patterns, know which parts to make deterministic, and design the loop that makes it reliable in production.

**Three devices carried through the whole book:**

1. **The four phases.** Every chapter hangs off one progression, **Prompt, Context, Harness, Loop**, the four phases the craft evolved through (Chapter 3). Each pattern chapter ends by naming which phase it belongs to.
2. **The running example — "Aegis," an incident-response agent.** A single system built up across the book: it triages production alerts, reasons over logs and dashboards, consults runbooks, proposes and (with approval) executes remediations, and learns from outcomes. Each pattern chapter adds one capability to Aegis, so the reader watches a real agent accrete from a prompt into a governed production system. Contrasting "big-name" case studies (Spotify Honk, ServiceNow, and others) appear alongside to show the same pattern at scale.
3. **"Design Debate" sidebars.** The major platforms have converged on architecture but openly disagree on philosophy — and those disagreements teach better than any single vendor's answer. Recurring one-page sidebars stage the five live debates at the chapter where each bites hardest, present both camps fairly (important given the author's Google affiliation — the book stays vendor-neutral), and leave the reader equipped to decide:
   - **Reach for multi-agent, or resist it?** (Google/Microsoft/AWS lean in; Anthropic/Cognition say start single-agent) → Chapter 13.
   - **How much of the loop to own?** (own the loop vs. govern the loop; a ladder every vendor sells end to end, from raw API to managed runtime) → Chapter 2, revisited Chapter 10.
   - **Where do guardrails live?** (IAM/policy vs. middleware pipeline vs. model/harness) → Chapter 20.
   - **Own-the-stack vs. framework-agnostic?** (Google's integrated bet vs. AWS/Microsoft "host any framework") → Chapter 21.
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
- **The failure** *(2–4 chapters only)* — a concrete way agents break without this pattern. Save it for where the failure is genuinely visceral: **Policy and Guardrails (Ch 20)**, **Loop Validation (Ch 14)**, and one of **Evaluation (Ch 9)** or **The New Bottlenecks (Ch 25)**.
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
- **Approval gates by design;** asymmetric checkpoints on high-stakes actions.
- **Outcome-based accountability:** review the change, not the author.
- **Native trigger surfaces** (Slack, GitHub, the tools people already use).
- Designing the interruption: how to pause, serialize state, and resume safely.
- *Aegis:* auto-diagnose freely; require human approval before any write to production.
- *In the wild:* Shopify Sidekick's approval gates; Block's confirmation on financial/prod actions; Honk landing work as reviewable PRs.
- *When not to use it:* low-stakes, reversible actions where a gate just adds latency.

### Chapter 9. Evaluation Patterns
- **Behavioral evaluation** beyond unit/integration tests: "what did it decide, and was it right?"
- **LLM-as-judge, validated against humans;** trusting the judge only once it matches evaluators.
- **Treat agents like regulated systems:** sandbox, simulate, then grant autonomy.
- **A/B against the prior baseline;** offline eval sets and regression suites for agents.
- Building the eval harness before you need it.
- *Aegis:* a simulation harness replaying historical incidents; an LLM judge scoring diagnosis quality.
- *In the wild:* Shopify's human + LLM-judge pre-deployment eval; Spotify's LLM eval judges for relevance/coherence/quality.
- *Vendor lens:* AWS AgentCore ships 13 pre-built evaluators as a platform component; OpenAI's AgentKit adds datasets, trace grading, and automated prompt optimization; Microsoft/Google bake evaluation into the managed runtime — evidence that eval is becoming platform infrastructure, not a bolt-on.
- *When not to use it:* (trick chapter — you always evaluate; the question is how much, not whether.)

---

# Part III — Composition (Loop Engineering)

*This is the heart of the book. Part II built the pieces; here we wire them into working loops. Loop engineering, the fourth phase from Chapter 3, gets its full treatment across this part: first make as much as possible deterministic, then introduce controlled non-determinism, coordinate agents, validate the loop, and finally let it improve itself.*

### Chapter 10. Loop Engineering
- Reprise the four phases: the pieces are built; composition is the act of closing them into loops.
- **The loop as the unit of reliability:** plan → act → observe → validate → correct → repeat-or-exit.
- Loop shapes: the single-agent reason-act loop, the orchestrated multi-agent loop, the human-gated loop.
- **Control and termination:** exit conditions, iteration ceilings, cost/latency budgets, and preventing runaway loops.
- Why the rest of Part III is "loop engineering": every chapter here is about closing one kind of loop well.
- *Aegis:* the top-level triage loop the agent runs on every alert.
- *Which phase:* this is the **Loop** phase from Chapter 3 (prompt, context, harness, loop), expanded into a discipline.

### Chapter 11. Deterministic Workflows
- **Start here on purpose:** the safe, predictable foundation before any probabilistic reasoning enters.
- Codifying repeatable, high-assurance steps: pipelines, state machines, DAGs.
- **Durable execution:** stateful, resumable, checkpointed workflows as the backbone.
- Reliability primitives: idempotency, retries, compensation/rollback (saga pattern).
- The principle: make as much of the system as possible *boringly predictable* — spend non-determinism only where it earns its keep.
- *Aegis:* the remediation runbook as a deterministic, resumable workflow with rollback.
- *In the wild:* the common pattern of a deterministic, heuristics-based planner or optimizer sitting behind LLM reasoning.
- *When not to use it:* genuinely novel or ambiguous tasks that can't be pre-scripted — which is what the next chapter is for.

### Chapter 12. The Non-Deterministic Split
- **The most consequential decision in the book:** which steps stay deterministic, which get adaptive LLM reasoning.
- **Don't agentify everything; don't build a giant rules engine either** — design the *hybrid*.
- **Cognitive routing + strict validation:** intelligence at the edges, determinism in the core; deterministic scaffold with LLM-filled gaps.
- **Bounding non-determinism:** wrap every probabilistic step in schemas, validators, retries, and deterministic guardrails.
- Patterns: router → specialists → deterministic executor.
- *Aegis:* the LLM diagnoses the incident (non-deterministic), but a deterministic validator must pass before it can trigger the runbook.
- *In the wild:* the recurring hybrid, LLM decides *what* to do while a deterministic engine guarantees *how well*, with teams rejecting both "more hard-coded workflows" and "one giant rules engine."
- *Vendor lens:* this split is the industry's central architectural claim — Microsoft's Agent Framework hard-separates **Agents** (stateful, probabilistic) from **Workflows** (graph-based, deterministic); AWS exposes deterministic **Policy** controls as a distinct AgentCore component; Google's customer framing is "deterministic business rules with probabilistic reasoning." Different words, same doctrine.
- *When not to use non-determinism:* high-assurance or regulated paths that can't tolerate variance.

### Chapter 13. Multi-Agent Orchestration
- When multiple agents are actually warranted (combinatorial workflows) — and when they aren't.
- Specialist roles: router, retrieval, action, validation agents.
- The plan–do–evaluate loop as the orchestration core.
- Orchestration frameworks and the A2A protocol for agent-to-agent collaboration.
- The contrarian rule: **avoid multi-agent early;** reach for sub-agents and low-level tools first.
- Failure modes: coordination overhead, latency, drift, debugging across agents.
- *Aegis:* when to split diagnosis and remediation into separate agents — and why you might not.
- *Vendor lens:* Microsoft names five reusable orchestration topologies (Sequential, Concurrent, Handoff, Group Chat, Magentic); AWS Bedrock Agents coordinate specialists under a supervisor; Google/A2A standardize cross-platform agent hand-offs.
- *Design Debate (multi-agent):* the field's sharpest live disagreement — Google/Microsoft/AWS ship multi-agent as a headline feature, while Anthropic and Cognition argue *start single-agent* because multi-agent fragments context and disperses decisions. Present both; let the reader's task decide.

### Chapter 14. Loop Validation
- Validating the loop itself: is each iteration making *progress*, and is the output *trustworthy* enough to proceed?
- **Verification inside the loop:** the agent checks its own work before it commits or advances.
- Self-correction and reflection loops; critic/validator agents; guardrail checks between steps.
- **Progress and stall detection:** catching no-progress loops, oscillation, and cost-aware early exit.
- Ground-truth checks vs. LLM-judge checks *inside* the loop (ties to the eval harness in Chapter 9).
- *Aegis:* MCP verification tools confirm a remediation actually cleared the alert before the loop closes the incident.
- *In the wild:* Spotify Honk's verification loops through MCP tools that check the work before a PR is raised.
- *When not to use it:* trivial single-shot tasks where validation costs more than the risk it removes.

### Chapter 15. Self-Evolving Agents
- From static loops to loops that improve themselves over time.
- **Mechanisms:** memory consolidation (episodic → procedural, from Chapter 6), skill/tool acquisition, self-reflection updates, learned routing.
- Feedback-to-improvement pipelines: turning outcomes and traces into better behavior.
- **Guardrails on self-modification:** bounded autonomy, human review of learned changes, and preventing drift or reward-hacking.
- The safety valve: **no learned change ships unevaluated** (ties to Chapter 9) — evaluation gates every promotion.
- *Aegis:* recurring incident resolutions get promoted into new runbooks automatically — but only past an eval gate and a human sign-off.
- *In the wild:* the field's push toward agents that learn from experience, with the open caution about stability and auditability.
- *When not to use it:* when auditability and stability outweigh adaptation — most regulated environments.

### Chapter 16. Putting It All Together — The Reference Architecture
- The full layered stack assembled from Parts II–III (the master diagram); Part IV then adds the hosting, isolation, observability, and policy layer around it.
- The design doctrine as a checklist: build for autonomy; reliability in the loop; LLM reasons / tools ground / deterministic core guarantees; curate context; guardrails in IAM; review outcomes; don't agentify everything.
- A design walkthrough: taking Aegis from blank page to production-ready using the whole framework.
- A one-page architecture review checklist the reader can apply at work.

---

# Part IV — Hosting Agents

*You've built the agent (Part II) and composed its behavior into working loops (Part III). Now run it for real. This part is the operational substrate: where an agent is hosted, how it's isolated, how it proves who it is and finds what it's allowed to use, how you watch what it does, and the policy that governs what it can do. It's the production layer the case studies in Part V lean on, and the difference between a demo and something you'd deploy.*

### Chapter 17. Identity and Discovery
- **Runtime capability discovery:** agents query a registry for approved tools and APIs instead of hard-coding endpoints; MCP registries.
- **Machine-readable capability specs** (e.g. Arazzo) so an agent can reason about what a capability does before it calls it.
- **Verifiable agent identity:** who the agent authenticates as; SPIFFE/mTLS between nodes; A2A Signed Agent Cards; agents as first-class principals, not static tokens.
- **Cashing out what Composition assumed:** the multi-agent patterns in Part III assumed agents could prove who they are and discover each other (the "when agents work together" layer from Chapter 2). This chapter is where that assumption becomes real infrastructure.
- *Aegis:* discovers only the remediation tools it's authorized for; authenticates as itself, so every action is attributable.
- *In the wild:* MCP registries; A2A v1.0 identity; the 2026 push toward verifiable agent credentials.
- *When not to use it:* a single, closed agent with a fixed toolset doesn't need a discovery layer.
- *Which phase:* harness.

### Chapter 18. Sandboxing and Runtime
- **Where agents run:** managed agent runtimes (Bedrock AgentCore Runtime, Vertex Agent Engine, Azure Foundry) vs. containers vs. serverless vs. self-hosted; the own-vs-govern-the-loop ladder (Chapter 2) applied to hosting.
- **Sandboxing and isolation:** running agent-generated code and risky tool calls in a sandbox (microVMs like Firecracker, gVisor, ephemeral containers, hosted code sandboxes); session isolation so one run can't see another; containing the blast radius.
- **Durable execution and state:** checkpointing, resumable long-running agents, state that survives a crash (the harness phase, Chapter 3).
- **Scaling the runtime:** horizontal scale, mostly-stateless workers, concurrency, and the cost of keeping them warm.
- *Aegis:* runs each remediation in a fresh sandbox; hosted on a managed runtime with durable state, so a long incident survives a restart.
- *In the wild:* Firecracker microVM isolation (AWS AgentCore Runtime); hosted code-execution sandboxes; durable-execution engines.
- *When not to use it:* a read-only agent that never executes code or writes state needs little of this.
- *Which phase:* harness.

### Chapter 19. Observability for Agents
- Beyond p95s and error budgets: **"what did the agent decide, and why?"**
- Capturing prompts, tool calls, intermediate decisions, and final outputs.
- Agentic metadata as an infrastructure layer; tracing across a multi-agent run.
- Turning traces into eval data (Chapter 9) and into the improvement signal for self-evolving loops (Chapter 15).
- *Aegis:* a decision trace for every incident the agent touches, feeding both the eval set and the runbook-promotion pipeline.
- *Vendor lens:* AWS crystallizes the core insight — traditional APM misses the bug that matters most, a *valid-but-wrong decision* in a multi-step workflow, so instrument the reasoning trace from the start (its AgentCore Observability spans four telemetry layers); OpenTelemetry has become the shared standard (Microsoft Agent Framework, AgentCore, and third-party tools like Arize/LangSmith/Langfuse all speak it).

### Chapter 20. Policy and Guardrails
- **Just-in-time authorization:** the agent decides at runtime what to call, so permissions are scoped dynamically, not provisioned up front.
- **Guardrails in IAM and policy, not the prompt:** "don't query the PII columns" belongs in access policy and config, where the model can't talk its way past it.
- **The social contract (from Chapter 2), enforced:** which tools it may reach for, which agents it may talk to, what it may see, all expressed as policy.
- **Content safety and injection defense:** input/output filtering, RAI middleware, indirect-prompt-injection defenses (Model Armor).
- **Blast-radius containment for chained execution:** preventing privilege escalation across autonomous tool chains; high-stakes actions routed to a human gate (ties to Human-in-the-Loop, Chapter 8).
- **Governance and audit:** every action attributable, logged, reviewable.
- *Aegis:* remediation gated by JIT authz; "never touch the customer DB" enforced in IAM; a human approval gate before any production write.
- *In the wild:* AWS AgentCore Identity/Policy; Google Model Armor + zero-trust; Microsoft's RAI middleware pipeline.
- *Design Debate (guardrail placement):* IAM/policy (AWS, Google) vs. middleware pipeline (Microsoft) vs. model/harness (Anthropic, OpenAI), the same guardrail, three homes.
- *When not to use it:* read-only, low-stakes agents where heavy policy machinery isn't warranted (but say so explicitly).
- *Which phase:* harness, plus the social contract.

---

# Part V — The Landscape and Case Studies

*A survey chapter, three scenario-driven design studies (each anchored on a different dominant force), and two real teardowns. The design studies are fresh systems, not Aegis, chosen to show the same patterns behaving differently when scale, the edge, or a merger is the thing that reorders every decision.*

### Chapter 21. The Vendor Landscape — Convergence and Its Discontents
- **The thesis:** the five major platforms have converged on nearly the same reference architecture (reasoning model + tools + memory + orchestration + identity + observability) and the same two protocols (MCP for tools, A2A for agents) — proof that Parts I–IV describe an industry consensus, not one opinion.
- **The convergent core** (the vendor-neutral reference model, mirrored from Chapter 16): deterministic/probabilistic split; MCP + A2A; memory as a first-class layer; identity & security as foundations; built-in decision tracing; model-agnostic managed runtimes; simplicity-first.
- **Five instantiations, one page each** — how each platform expresses the same patterns:
  - **AWS** — Bedrock AgentCore (Runtime/Gateway/Memory/Identity/Observability/Policy/Evaluations); ops-first; "is this really an agent?" gate; the *valid-but-wrong decision* as the bug that matters.
  - **Anthropic** — *Building Effective Agents* (simplicity, transparency, agent-computer interface); context as a finite attention budget; reduce abstraction in production.
  - **Microsoft** — Agent Framework 1.0; the Agents-vs-Workflows separation as a foundational decision; five orchestration patterns; RAI middleware pipeline.
  - **Google** — Gemini Enterprise Agent Platform (ADK / Agent Studio / Agent Engine); A2A authorship; Model Armor + zero-trust; own-the-stack bet; deterministic rules + probabilistic reasoning.
  - **OpenAI** — Responses API → Agents SDK → AgentKit; the thin, model-hugging harness.
- **The independent frame:** Andrew Ng's four patterns (reflection, tool use, planning, multi-agent); LangGraph's graph model; Cognition's "don't build multi-agent" caution; the academic pattern catalogues — used to keep the survey above any vendor's marketing.
- **The five debates** (forward-referencing the Design Debate sidebars): multi-agent, abstraction, guardrail placement, own-stack vs. agnostic, code vs. visual.
- *Why this chapter is here, not in Part I:* the reader needs the patterns (Parts II–IV) before the vendor mapping means anything; placing it as the case-study opener turns "here's the landscape" into the natural runway for the studies that follow.

### Chapter 22. The Peak-Day Marketplace — Designing for Scale
- The dominant force: volume, throughput, cost, and latency at millions of requests. When scale is the constraint, it reorders every decision.
- **The use case:** the shopping assistant on a giant online marketplace, the kind Amazon or Alibaba run. A few questions per second on an ordinary Tuesday, then 50x that on Black Friday or Singles' Day, when every extra 100ms and every wasted cent per query is multiplied across tens of millions of concurrent shoppers.
- Design moves driven by scale: multi-model routing and cascades (a cheap model for the 90%, a stronger one only for the hard 10%, Chapter 4); aggressive Cacheable Prefix and batch APIs (Chapters 4, 5); deterministic fast paths for common cases (Chapters 11, 12); async and background work; horizontal scaling of a mostly-stateless harness with durable state only where needed; sampled observability.
- The trade-offs: freshness vs. cost, per-request routing overhead, where to spend the frontier budget.
- *In the wild:* **Klarna's** OpenAI-powered assistant is the flagship scale case: ~2.3M conversations in month one, roughly two-thirds of support chats, framed as the work of ~700 (later ~853) agents, across 23 markets and 35+ languages, resolution 11 min to under 2 min, cost-per-transaction $0.32 to $0.19; grounded in help-center content with low-confidence cases routed to humans. Use the **2025 walk-back** as the cautionary half: humans rehired for complex, emotional, and dispute cases, and weak escalation handoffs (customers re-explaining themselves) as the failure that hurt CSAT. Secondary: Rocket AI Agent on Amazon Bedrock (3x conversion, 85% fewer transfers).
- Patterns that dominate: multi-model, caching, deterministic workflows.

### Chapter 23. The Delivery Courier — Designing for the Edge
- The dominant force: intermittent or absent connectivity, latency, and constrained hardware (and privacy, for some edge cases). Speed and locality win.
- **The use case:** the courier's assistant inside a food-delivery app, the kind Uber Eats or DoorDash run. It lives on the driver's phone, guiding the route and handling order changes and customer messages, and it has to keep working when they drop into a parking garage, a tunnel, or a rural dead zone, so it can't assume the cloud is there.
- Design moves driven by the edge: small or fine-tuned open models on-device (Chapter 4); local tiered memory with sync-on-reconnect (Chapter 6); graceful degradation and a local fallback model; a hybrid split, a small local model that escalates to a cloud frontier model only when connected; actions that queue locally and reconcile when the signal returns.
- The trade-offs: capability vs. footprint, staleness of local memory, the sync-and-merge problem.
- *In the wild:* **DoorDash, Uber, and Grubhub** testing multi-step agentic ordering in the Google Gemini app (Mar 2026), where Gemini acts in-app inside an on-device "secure virtual window" (edge sandboxing). The dispatch brains are already edge-shaped: Uber's Trip State Model and DoorDash's DeepRed engine run on real-time GPS and motion-sensor data, and last-mile driver apps are built around offline sync and geofencing. The hybrid split is a live research line: on-device inference on NPU-class phones (small Gemma/Phi/Qwen variants) with edge-SLM-to-cloud-LLM escalation.
- Patterns that dominate: multi-model (edge/small), local tiered memory, fallbacks.

### Chapter 24. The Bank and the Neobank — Designing for a Merger
- The dominant force: heterogeneity, governance, and identity across two organizations, a large enterprise absorbing a small one, each with its own stack, data, and risk culture.
- **The use case:** a century-old retail bank acquires a fast-growing neobank. The bank's mainframe-era core and the startup's cloud-native agents have to serve the same customers from day one, across two identity systems, two risk cultures, and two data regimes, without a two-year migration first.
- Design moves driven by the merger: agents as the glue bridging legacy enterprise and modern startup systems via MCP connectors (Chapter 7); identity federation and memory Scoping across two orgs (Chapters 6, 20); guardrails reconciling two risk cultures (the "social contract" from Chapter 2); data residency and access control; incremental migration with agents carrying the seam through a multi-year integration.
- The trade-offs: autonomy vs. governance, whose conventions win, how much to unify vs. bridge.
- *In the wild:* **TELUS Digital's** agentic-banking write-up is this scenario almost verbatim: a backend-for-frontend (BFF) plus MCP orchestration layer gives agents permissioned access to a 40-year-old mainframe without replacing it ("build above it"). **Strata's Maverics** does M&A identity integration via an abstraction layer over two orgs' identity providers, translating protocols instead of migrating identities. **Deloitte** describes agents that reconcile records and catalogs across legacy systems and flag conflicts, avoiding an immediate data migration. War story: a post-merger ERP consolidation (Western Digital, 25 legacy systems retired) using scout agents to map shadow processes from transaction logs. Strong enough that Ch 24 could run as a light teardown, not a pure hypothetical.
- Patterns that dominate: tooling/MCP, scoping, identity and guardrails.

### Chapter 25. Spotify Honk — A Coding Agent, Torn Down
- A background coding agent powered by Claude, built on the existing Fleet Management harness.
- Slack-native trigger; MCP-exposed verification tools; feedback loops for predictable results.
- Context engineering across thousands of repos; loop engineering as the reliability source.
- Outcome-based accountability; PRs as the human-review gate.
- Result: 1,500+ merged PRs; the shift from code *creation* to full-lifecycle work.

### Chapter 26. ServiceNow — The Enterprise Workflow Agent
- The scenario: agentic IT and enterprise-workflow automation (incident, request, and change management) across a large enterprise's connected systems.
- Architecture: LLM reasoning wrapped around deterministic, governed workflows; connected/specialized agents coordinating over enterprise data; approvals and audit built in.
- Why it's the counterweight to Honk: heavily governed, deterministic-first, and enterprise-integration-heavy, where Honk is developer-autonomous and loop-first.
- Lessons: deterministic workflow plus probabilistic reasoning; governance as a first-class design input; the enterprise-integration reality most production agents actually live in.
- *(Research current ServiceNow AI Agents / Now Assist specifics before drafting.)*

---

# Part VI — Frontier

### Chapter 27. The New Bottlenecks
- What breaks when agents ship work faster than humans can review it.
- Accountability, trust collapse, and the review-throughput problem.
- Agent "factories": coordinated agents producing complex knowledge work.

### Chapter 28. The Full Lifecycle
- Beyond creation: maintenance, migration, deletion — the work nobody wants.
- Agent-first platforms (developer portals becoming agent interfaces via MCP).
- Edge and alternative-cloud inference: proximity over raw processing power.

### Chapter 29. Identity, Trust, and the Agentic Web
- Machine identity as a top-tier security concern.
- Verifiable credentials, signed agent cards, and the non-human internet.
- Open standards (MCP, A2A) and where the ecosystem is heading.

---

# Appendices

- **A. The chapter-pattern quick reference** — every pattern on one page.
- **B. The Aegis codebase** — the running example, assembled end to end.
- **C. Architecture review checklist** — the Chapter 16 checklist, printable.
- **D. Framework & protocol cheat sheet** — ADK, LangGraph, CrewAI, Bedrock, MCP, A2A, Arazzo.
- **E. Glossary** — agentic misalignment, JIT authz, agentic metadata, plan-do-evaluate, and more.

---

## Notes on sequencing (for the author)

- **Parts I–IV are the backbone of the argument; Part V is proof; Part VI is the horizon.** A reader could stop after Chapter 16 for the complete design method, or after Part IV (Chapter 20) for the production-ready one.
- **Aegis must be introduced in Chapter 1 and touched in every pattern chapter** or the "cumulative" feel is lost, this is the highest-risk thread to maintain.
- **Chapter 3 (the four phases) and the loop-engineering cluster (Chapters 10–12) are what sell the book.** If a reviewer reads only those, they should get the whole thesis.
- **Part III's internal order (Composition) is deliberate:** deterministic first (Ch 11), then the split (Ch 12), so the reader anchors on predictability before non-determinism. Loop validation (Ch 14) follows multi-agent (Ch 13) because validating a single loop is a prerequisite to trusting many. Self-evolution (Ch 15) closes the part by planting a need Part IV answers: you can't safely evolve what you can't observe, so observability (Ch 19) now lands as the operational concern it is, in Hosting Agents.
- **Part V mixes scenario design studies and teardowns.** Chapters 22–24 are fresh, Aegis-free system designs, each anchored on one dominant force (scale, edge, merger) to show the same patterns behaving differently under different pressure. Chapters 25–26 are real teardowns (Honk, ServiceNow) chosen as opposites: developer-autonomous and loop-first (Honk) vs. governed and deterministic-first (ServiceNow). Spotify Ads AI was cut; Shopify and Block survive only as in-text vendor-lens examples in Part II. The security split and the new design studies grew the book to 29 chapters, so watch the page budget.
- **The Design Debate sidebars are the neutrality mechanism.** Five recurring sidebars (mapped in "About this book") stage the real vendor disagreements at the chapters where they bite, presenting both camps rather than an answer. Given the author's Google affiliation, holding these scrupulously even-handed is what protects the book's vendor-neutral credibility — Google appears as one instantiation among five, never the default.
- **The vendor landscape (Ch 21) is one chapter, not a part — deliberately.** Instead of a chapter per platform (which bloats the page count and dates fastest), the convergence table is the single reference model and each vendor is a one-page instantiation. This is the main lever for hitting the ~500-page target while staying current.
- Keep case-study *and vendor* specifics (versions, metrics, product names) dated and quarantined in Part V and the appendices — branding shifts fast (e.g., Vertex AI Agent Builder → Gemini Enterprise Agent Platform within a single year), so the pattern chapters in Parts I–IV must stay product-free to age well.
- **Voice consistency is a first-class risk, like the Aegis thread.** Draft the *voiced* sections (openers, sidebars, closes) against the Voice & Style Guide, and resist letting the voice creep into the mechanics. If a chapter opener doesn't have a real scar or scene behind it, write it plainer rather than manufacturing an anecdote — a forced war story reads worse than none.

### Sources underpinning the examples
- InfoWorld, *Best practices for building agentic systems* (Apr 2026).
- Spotify Engineering: *Spotify x Anthropic Live* + Honk series (Nov 2025–Apr 2026) for the Chapter 25 teardown. ServiceNow AI Agents / Now Assist for Chapter 26 (research current specifics before drafting).
- IBM Think 2026 recap; A2A protocol v1.0; 2026 MCP roadmap.
- Vendor landscape (Ch 21): AWS Bedrock AgentCore docs & AWS ML blog; Anthropic *Building Effective Agents* (Dec 2024) and *Effective Context Engineering for AI Agents* (Sept 2025); Microsoft *Agent Framework 1.0* (GA Apr 2026) & Azure Architecture Center; Google Cloud Next 2026 / Gemini Enterprise Agent Platform docs; OpenAI *AgentKit* & Agents SDK; Andrew Ng (AI Ascent 2024); Cognition "Don't Build Multi-Agents"; Liu et al., *Agent Design Pattern Catalogue* (JSS 2025). *(Full detail in the companion vendor-landscape briefing.)*

### Design-study case sources (Ch 22–24) — for the author
*Recent online cases gathered to anchor the three design studies. Volatile: verify figures and check for newer coverage before drafting.*

- **Ch 22 (Scale) — Klarna AI assistant:**
  - Klarna press release, *AI assistant handles two-thirds of customer service chats in its first month* — klarna.com/international/press/klarna-ai-assistant-handles-two-thirds-of-customer-service-chats-in-its-first-month/
  - LangChain case study (built on LangGraph/LangSmith), Jun 2026 — langchain.com/blog/customers-klarna
  - CX Dive, cost-per-transaction $0.32→$0.19, May 2025 — customerexperiencedive.com/news/klarna-ai-slash-customer-service-costs/748647/
  - Twig, deep analysis incl. the 2025 walk-back — twig.so/blog/klarna-ai-customer-support-efficiency
  - ZenML LLMOps DB (architecture summary; also Rocket AI Agent on Bedrock) — zenml.io/llmops-database/ai-assistant-for-global-customer-service-automation

- **Ch 23 (Edge) — food-delivery / on-device:**
  - Stellagent recap of Retail TouchPoints: DoorDash/Uber/Grubhub agentic ordering in Gemini; on-device "secure virtual window", Mar 2026 — stellagent.ai/insights/doordash-uber-gemini-agentic-ordering
  - Riseup Labs: Uber Trip State Model, DoorDash DeepRed dispatch, DashAI, Jan 2026 — riseuplabs.com/ai-in-food-delivery-app-development/
  - Bolder Apps: on-device inference, NPUs, AI-native mobile apps, 2026 — bolderapps.com/blog-posts/beyond-the-chatbot-5-ways-ai-native-mobile-apps-are-disrupting-traditional-service-industries-in-2026
  - arXiv survey: *Collaborative Inference and Learning between Edge SLMs and Cloud LLMs* (2507.16731) — the edge/cloud hybrid split

- **Ch 24 (Merger) — M&A integration:**
  - TELUS Digital: *Agentic banking without core replacement* (BFF + MCP bridge over the mainframe), May 2026 — telusdigital.com/insights/data-and-ai/article/agentic-banking-without-core-replacement
  - Strata.io: Maverics identity orchestration for M&A (abstraction layer over IDPs), Apr 2026 — strata.io/blog/app-identity-modernization/managing-identity-in-a-merger-acquisition-or-divestiture/
  - Deloitte: *Where is the value of AI in M&A* (multi-agent + modern data architecture; reconciliation agents) — deloitte.com/cz-sk/en/services/consulting/blogs/where-is-the-value-of-AI-in-MA-why-multi-agent-systems-needs-modern-data-architecture.html
  - CIO / Puneet Thakkar: *From go-live to always-live* (scout/user-proxy agents; Western Digital ERP consolidation), Feb 2026 — cio.com/article/4128116/from-go-live-to-always-live-how-agentic-ai-is-rewriting-the-finance-transformation-playbook.html
  - Accenture: *Agentic AI reshaping M&A*, May 2026 — accenture.com/us-en/insights/strategy/agentic-ai-reshaping-mergers-and-acquisitions
  - Forbes: *How AI speeds up post-merger IT integration*, May 2026 — forbes.com/sites/cio/2026/05/21/how-ai-speeds-up-post-merger-it-integration/
