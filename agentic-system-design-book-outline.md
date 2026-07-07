# Agentic System Design
### Patterns for Engineers and Architects Building Autonomous Systems

*Full chapter-by-chapter outline / table of contents (working draft)*

---

## About this book

**Audience:** engineers and architects designing, building, and operating agentic systems — people who will write the harness, wire the tools, and get paged when the agent does something surprising.

**Promise:** by the end, the reader can look at any agent requirement and decompose it into the right patterns, know which parts to make deterministic, and design the loop that makes it reliable in production.

**Three devices carried through the whole book:**

1. **The four phases.** Every chapter hangs off one progression, **Prompt, Context, Harness, Loop**, the four phases the craft evolved through (Chapter 3). Each pattern chapter ends by naming which phase it belongs to.
2. **The running example — "Aegis," an incident-response agent.** A single system built up across the book: it triages production alerts, reasons over logs and dashboards, consults runbooks, proposes and (with approval) executes remediations, and learns from outcomes. Each pattern chapter adds one capability to Aegis, so the reader watches a real agent accrete from a prompt into a governed production system. Contrasting "big-name" case studies (Spotify Ads AI, Spotify Honk, Shopify, Block, ServiceNow) appear alongside to show the same pattern at scale.
3. **"Design Debate" sidebars.** The major platforms have converged on architecture but openly disagree on philosophy — and those disagreements teach better than any single vendor's answer. Recurring one-page sidebars stage the five live debates at the chapter where each bites hardest, present both camps fairly (important given the author's Google affiliation — the book stays vendor-neutral), and leave the reader equipped to decide:
   - **Reach for multi-agent, or resist it?** (Google/Microsoft/AWS lean in; Anthropic/Cognition say start single-agent) → Chapter 14.
   - **How much of the loop to own?** (own the loop vs. govern the loop; a ladder every vendor sells end to end, from raw API to managed runtime) → Chapter 2, revisited Chapter 11.
   - **Where do guardrails live?** (IAM/policy vs. middleware pipeline vs. model/harness) → Chapter 8.
   - **Own-the-stack vs. framework-agnostic?** (Google's integrated bet vs. AWS/Microsoft "host any framework") → Chapter 19.
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
- **The failure** *(2–4 chapters only)* — a concrete way agents break without this pattern. Save it for where the failure is genuinely visceral: **Discoverability & Security (Ch 8)**, **Loop Validation (Ch 15)**, and one of **Evaluation (Ch 10)** or **The New Bottlenecks (Ch 23)**.
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
- *Design Debate (how much of the loop to own):* own the loop vs. govern the loop, a ladder each vendor sells end to end (Gemini API, to ADK, to Antigravity SDK, to managed platform). Introduced here, revisited Chapter 11.

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
- *In the wild:* Spotify Ads AI's RouterAgent (cheap routing) feeding specialized resolvers; harnesses that send simple steps to small models and hard steps to large ones.
- *When not to use it:* a single capable model is simpler; do not add routing until cost or latency forces it, and do not fine-tune unless the task is narrow, stable, and high-volume enough to repay the MLOps cost (a frontier API is the right default for broad, changing, or low-volume work).
- *Which phase:* harness (routing is a harness job), with a foot in the loop (escalation is a feedback decision).

### Chapter 5. Prompt & Context Patterns
- **Prompt-as-code:** version control, tests, iteration; guardrails at prompt *and* parse layers.
- **Context engineering, kept simple:** reproducible setups, `CLAUDE.md`-style files, reusable domain "skills."
- **Just-in-time context** and progressive disclosure over a bloated system prompt.
- Semantic nuance: encoding domain meaning so the agent isn't confidently wrong.
- Token economics: the cost of over-stuffing context.
- *Aegis:* a skills file per service domain; retrieving the relevant runbook only when the alert type is known.
- *In the wild:* Spotify/Anthropic on context engineering at thousands-of-repos scale.
- *Vendor lens (Anthropic):* context as a *finite attention budget* — find the smallest set of high-signal tokens; keep system prompts at the "right altitude" (between brittle hardcoded logic and vague hand-waving); production techniques worth their own sections — compaction, just-in-time retrieval, memory tools (CRUD memory files), and programmatic tool calling (code orchestrates tools so only the final result re-enters context). Microsoft's angle: decide per agent-transition whether the next agent needs full, compacted, or no accumulated context.
- *When not to use it:* over-engineering context for a narrow, stable task.

### Chapter 6. Memory Patterns
- **Two lenses on memory.** Separate *what kind of knowledge* is being held (the cognitive taxonomy below) from *where it physically lives* (the storage tiers). They're orthogonal — the same episodic memory might sit in a cache today and a vector store tomorrow.

- **The four types of memory (by kind of knowledge):**
  - **Working memory** — the here-and-now scratchpad: the active task, the current reasoning chain, the in-flight context window. Volatile, small, discarded when the task ends. This is what the agent is *thinking about right now*.
  - **Episodic memory** — records of *what happened*: past interactions, prior runs, specific events and their outcomes ("on 2026-03-14, this alert was caused by X and fixed by Y"). Time-stamped, instance-level, the raw material for learning from experience.
  - **Semantic memory** — general *facts and knowledge*: domain concepts, entities, policies, documentation, the meaning of terms. Not tied to a single event. Typically the vector store + knowledge graph / catalog layer.
  - **Procedural memory** — *how to do things*: learned skills, tool-use policies, runbooks, and the "muscle memory" of which action sequence works for which situation. Often encoded as skills files, workflow definitions, or fine-tuned behavior rather than retrieved text.

- **Mapping types → storage tiers:** working memory lives in the context window / session state; episodic and semantic memory in long-term stores (vector DB for recall, structured store for precision); procedural memory in skills/workflow definitions and hot caches. Choosing the store per type is the real design work.
- Curation over volume: output quality tracks context quality — retrieve the *relevant* episode or fact, not all of them.
- Memory hygiene: staleness, invalidation, consolidation (promoting a repeated episode into a semantic fact or a procedure), and the cost of remembering the wrong thing.
- *Aegis mapped across all four:* **working** = the state of the incident it's currently triaging; **episodic** = the log of past incidents and how each was resolved; **semantic** = service ownership, dependency graph, and runbook *knowledge*; **procedural** = the learned remediation playbooks it executes. Watch consolidation in action: three similar incidents (episodic) become a new runbook (procedural).
- *In the wild:* Spotify Ads AI's persistent session store (working/session) + in-memory historical-performance cache (episodic recall) + PostgreSQL system of record.
- *When not to use it:* stateless tasks where memory adds risk without value — and specifically, resist adding episodic/procedural memory before you can evaluate whether it improves outcomes (Chapter 10).

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

### Chapter 8. Discoverability & Security
- **Runtime capability discovery:** registries (MCP registries) instead of hard-coded endpoints.
- **Machine-readable workflow specs** (e.g. Arazzo) for multi-step behaviors.
- **Just-in-time authorization:** why you can't scope agent permissions the traditional way.
- **Guardrails in IAM, not the prompt:** policy and config over prompt instructions.
- **Verifiable agent identity:** SPIFFE/mTLS, A2A Signed Agent Cards; agents as first-class principals.
- **Containing blast radius:** permissions for chained, autonomous execution.
- *Aegis:* the agent authenticates as itself; remediation tools gated by JIT authz; "never touch the customer DB" enforced in IAM.
- *In the wild:* MCP registries + A2A v1.0 identity; the practitioner rule that guardrails live in IAM.
- *Vendor lens:* Google's Model Armor (indirect prompt-injection defense) + zero-trust applied to decentralized agents + IAM/audit logging; AWS's AgentCore Identity (OAuth/IAM) as a first-class component; Microsoft's middleware pipeline injecting content-safety and compliance checks into the execution loop.
- *Design Debate (guardrail placement):* IAM/policy (AWS, Google) vs. middleware pipeline (Microsoft) vs. model/harness (Anthropic, OpenAI) — the same guardrail, three homes.
- *When not to use it:* read-only, low-stakes agents where heavy identity machinery isn't warranted (but say so explicitly).

### Chapter 9. Human-in-the-Loop Patterns
- **Approval gates by design;** asymmetric checkpoints on high-stakes actions.
- **Outcome-based accountability:** review the change, not the author.
- **Native trigger surfaces** (Slack, GitHub, the tools people already use).
- Designing the interruption: how to pause, serialize state, and resume safely.
- *Aegis:* auto-diagnose freely; require human approval before any write to production.
- *In the wild:* Shopify Sidekick's approval gates; Block's confirmation on financial/prod actions; Honk landing work as reviewable PRs.
- *When not to use it:* low-stakes, reversible actions where a gate just adds latency.

### Chapter 10. Evaluation Patterns
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

### Chapter 11. Loop Engineering
- Reprise the four phases: the pieces are built; composition is the act of closing them into loops.
- **The loop as the unit of reliability:** plan → act → observe → validate → correct → repeat-or-exit.
- Loop shapes: the single-agent reason-act loop, the orchestrated multi-agent loop, the human-gated loop.
- **Control and termination:** exit conditions, iteration ceilings, cost/latency budgets, and preventing runaway loops.
- Why the rest of Part III is "loop engineering": every chapter here is about closing one kind of loop well.
- *Aegis:* the top-level triage loop the agent runs on every alert.
- *Which phase:* this is the **Loop** phase from Chapter 3 (prompt, context, harness, loop), expanded into a discipline.

### Chapter 12. Deterministic Workflows
- **Start here on purpose:** the safe, predictable foundation before any probabilistic reasoning enters.
- Codifying repeatable, high-assurance steps: pipelines, state machines, DAGs.
- **Durable execution:** stateful, resumable, checkpointed workflows as the backbone.
- Reliability primitives: idempotency, retries, compensation/rollback (saga pattern).
- The principle: make as much of the system as possible *boringly predictable* — spend non-determinism only where it earns its keep.
- *Aegis:* the remediation runbook as a deterministic, resumable workflow with rollback.
- *In the wild:* Spotify Ads AI's deterministic, heuristics-based MediaPlanner engine.
- *When not to use it:* genuinely novel or ambiguous tasks that can't be pre-scripted — which is what the next chapter is for.

### Chapter 13. The Non-Deterministic Split
- **The most consequential decision in the book:** which steps stay deterministic, which get adaptive LLM reasoning.
- **Don't agentify everything; don't build a giant rules engine either** — design the *hybrid*.
- **Cognitive routing + strict validation:** intelligence at the edges, determinism in the core; deterministic scaffold with LLM-filled gaps.
- **Bounding non-determinism:** wrap every probabilistic step in schemas, validators, retries, and deterministic guardrails.
- Patterns: router → specialists → deterministic executor.
- *Aegis:* the LLM diagnoses the incident (non-deterministic), but a deterministic validator must pass before it can trigger the runbook.
- *In the wild:* Spotify Ads AI — LLM decides *what* to plan; the deterministic engine guarantees *how well* — and the team's explicit rejection of both "more hard-coded workflows" and "one giant rules engine."
- *Vendor lens:* this split is the industry's central architectural claim — Microsoft's Agent Framework hard-separates **Agents** (stateful, probabilistic) from **Workflows** (graph-based, deterministic); AWS exposes deterministic **Policy** controls as a distinct AgentCore component; Google's customer framing is "deterministic business rules with probabilistic reasoning." Different words, same doctrine.
- *When not to use non-determinism:* high-assurance or regulated paths that can't tolerate variance.

### Chapter 14. Multi-Agent Orchestration
- When multiple agents are actually warranted (combinatorial workflows) — and when they aren't.
- Specialist roles: router, retrieval, action, validation agents.
- The plan–do–evaluate loop as the orchestration core.
- Orchestration frameworks and the A2A protocol for agent-to-agent collaboration.
- The contrarian rule: **avoid multi-agent early;** reach for sub-agents and low-level tools first.
- Failure modes: coordination overhead, latency, drift, debugging across agents.
- *Aegis:* when to split diagnosis and remediation into separate agents — and why you might not.
- *Vendor lens:* Microsoft names five reusable orchestration topologies (Sequential, Concurrent, Handoff, Group Chat, Magentic); AWS Bedrock Agents coordinate specialists under a supervisor; Google/A2A standardize cross-platform agent hand-offs.
- *Design Debate (multi-agent):* the field's sharpest live disagreement — Google/Microsoft/AWS ship multi-agent as a headline feature, while Anthropic and Cognition argue *start single-agent* because multi-agent fragments context and disperses decisions. Present both; let the reader's task decide.

### Chapter 15. Loop Validation
- Validating the loop itself: is each iteration making *progress*, and is the output *trustworthy* enough to proceed?
- **Verification inside the loop:** the agent checks its own work before it commits or advances.
- Self-correction and reflection loops; critic/validator agents; guardrail checks between steps.
- **Progress and stall detection:** catching no-progress loops, oscillation, and cost-aware early exit.
- Ground-truth checks vs. LLM-judge checks *inside* the loop (ties to the eval harness in Chapter 10).
- *Aegis:* MCP verification tools confirm a remediation actually cleared the alert before the loop closes the incident.
- *In the wild:* Spotify Honk's verification loops through MCP tools that check the work before a PR is raised.
- *When not to use it:* trivial single-shot tasks where validation costs more than the risk it removes.

### Chapter 16. Self-Evolving Agents
- From static loops to loops that improve themselves over time.
- **Mechanisms:** memory consolidation (episodic → procedural, from Chapter 6), skill/tool acquisition, self-reflection updates, learned routing.
- Feedback-to-improvement pipelines: turning outcomes and traces into better behavior.
- **Guardrails on self-modification:** bounded autonomy, human review of learned changes, and preventing drift or reward-hacking.
- The safety valve: **no learned change ships unevaluated** (ties to Chapter 10) — evaluation gates every promotion.
- *Aegis:* recurring incident resolutions get promoted into new runbooks automatically — but only past an eval gate and a human sign-off.
- *In the wild:* the field's push toward agents that learn from experience, with the open caution about stability and auditability.
- *When not to use it:* when auditability and stability outweigh adaptation — most regulated environments.

### Chapter 17. Observability for Agents
- Beyond p95s and error budgets: **"what did the agent decide, and why?"**
- Capturing prompts, tool calls, intermediate decisions, and final outputs.
- Agentic metadata as an infrastructure layer; tracing across a multi-agent run.
- Turning traces into eval data (Chapter 10) and into the improvement signal for self-evolving loops (Chapter 16).
- *Aegis:* a decision trace for every incident the agent touches, feeding both the eval set and the runbook-promotion pipeline.
- *Vendor lens:* AWS crystallizes the core insight — traditional APM misses the bug that matters most, a *valid-but-wrong decision* in a multi-step workflow, so instrument the reasoning trace from the start (its AgentCore Observability spans four telemetry layers); OpenTelemetry has become the shared standard (Microsoft Agent Framework, AgentCore, and third-party tools like Arize/LangSmith/Langfuse all speak it).

### Chapter 18. Putting It All Together — The Reference Architecture
- The full layered stack assembled from Parts II–III (the master diagram).
- The design doctrine as a checklist: build for autonomy; reliability in the loop; LLM reasons / tools ground / deterministic core guarantees; curate context; guardrails in IAM; review outcomes; don't agentify everything.
- A design walkthrough: taking Aegis from blank page to production-ready using the whole framework.
- A one-page architecture review checklist the reader can apply at work.

---

# Part IV — The Landscape & Case Studies (Teardowns)

### Chapter 19. The Vendor Landscape — Convergence and Its Discontents
- **The thesis:** the five major platforms have converged on nearly the same reference architecture (reasoning model + tools + memory + orchestration + identity + observability) and the same two protocols (MCP for tools, A2A for agents) — proof that Parts I–III describe an industry consensus, not one opinion.
- **The convergent core** (the vendor-neutral reference model, mirrored from Chapter 18): deterministic/probabilistic split; MCP + A2A; memory as a first-class layer; identity & security as foundations; built-in decision tracing; model-agnostic managed runtimes; simplicity-first.
- **Five instantiations, one page each** — how each platform expresses the same patterns:
  - **AWS** — Bedrock AgentCore (Runtime/Gateway/Memory/Identity/Observability/Policy/Evaluations); ops-first; "is this really an agent?" gate; the *valid-but-wrong decision* as the bug that matters.
  - **Anthropic** — *Building Effective Agents* (simplicity, transparency, agent-computer interface); context as a finite attention budget; reduce abstraction in production.
  - **Microsoft** — Agent Framework 1.0; the Agents-vs-Workflows separation as a foundational decision; five orchestration patterns; RAI middleware pipeline.
  - **Google** — Gemini Enterprise Agent Platform (ADK / Agent Studio / Agent Engine); A2A authorship; Model Armor + zero-trust; own-the-stack bet; deterministic rules + probabilistic reasoning.
  - **OpenAI** — Responses API → Agents SDK → AgentKit; the thin, model-hugging harness.
- **The independent frame:** Andrew Ng's four patterns (reflection, tool use, planning, multi-agent); LangGraph's graph model; Cognition's "don't build multi-agent" caution; the academic pattern catalogues — used to keep the survey above any vendor's marketing.
- **The five debates** (forward-referencing the Design Debate sidebars): multi-agent, abstraction, guardrail placement, own-stack vs. agnostic, code vs. visual.
- *Why this chapter is here, not in Part I:* the reader needs the patterns (Parts II–III) before the vendor mapping means anything; placing it as the case-study opener turns "here's the landscape" into the natural runway for the specific teardowns that follow.

### Chapter 20. Spotify Ads AI — A Hybrid Multi-Agent System
- The structural problem: fragmented decision logic across buying channels.
- Architecture: RouterAgent → parallel resolver agents → deterministic MediaPlanner.
- Stack: Google ADK, Vertex AI (Gemini 2.5 Pro), gRPC, in-memory cache.
- Trade-offs they made: single vs. multi-agent, in-memory vs. DB cache, sync vs. streaming.
- Results: 15–30 minutes → 5–10 seconds; 20+ fields → 1–3 messages.
- Lessons: prompt engineering *is* software engineering; agent boundaries matter; tools enable grounding.

### Chapter 21. Spotify Honk — Loop Engineering at Scale
- A background coding agent powered by Claude, built on the existing Fleet Management harness.
- Slack-native trigger; MCP-exposed verification tools; feedback loops for predictable results.
- Context engineering across thousands of repos.
- Outcome-based accountability; PRs as the human-review gate.
- Result: 1,500+ merged PRs; the shift from code *creation* to full-lifecycle work.

### Chapter 22. Shorter Studies
- **Shopify Sidekick:** human-in-the-loop by design; sub-agent architecture with low-level tools; the ~20–50 tool boundary.
- **Block (goose / Moneybot):** open-source agent tooling; MCP; human checkpoints on money and production.
- **ServiceNow:** IT incident resolution as coordinated, governed workflows across enterprise systems.
- Cross-study synthesis: what the winners share, where they diverge.

---

# Part V — Frontier

### Chapter 23. The New Bottlenecks
- What breaks when agents ship work faster than humans can review it.
- Accountability, trust collapse, and the review-throughput problem.
- Agent "factories": coordinated agents producing complex knowledge work.

### Chapter 24. The Full Lifecycle
- Beyond creation: maintenance, migration, deletion — the work nobody wants.
- Agent-first platforms (developer portals becoming agent interfaces via MCP).
- Edge and alternative-cloud inference: proximity over raw processing power.

### Chapter 25. Identity, Trust, and the Agentic Web
- Machine identity as a top-tier security concern.
- Verifiable credentials, signed agent cards, and the non-human internet.
- Open standards (MCP, A2A) and where the ecosystem is heading.

---

# Appendices

- **A. The chapter-pattern quick reference** — every pattern on one page.
- **B. The Aegis codebase** — the running example, assembled end to end.
- **C. Architecture review checklist** — the Chapter 18 checklist, printable.
- **D. Framework & protocol cheat sheet** — ADK, LangGraph, CrewAI, Bedrock, MCP, A2A, Arazzo.
- **E. Glossary** — agentic misalignment, JIT authz, agentic metadata, plan-do-evaluate, and more.

---

## Notes on sequencing (for the author)

- **Parts I–III are the backbone of the argument; Part IV is proof; Part V is the horizon.** A reader could stop after Chapter 18 and have a complete method.
- **Aegis must be introduced in Chapter 1 and touched in every Part II chapter** or the "cumulative" feel is lost, this is the highest-risk thread to maintain.
- **Chapter 3 (the four phases) and the loop-engineering cluster (Chapters 11–13) are what sell the book.** If a reviewer reads only those, they should get the whole thesis.
- **Part III's internal order is deliberate:** deterministic first (Ch 12), then the split (Ch 13), so the reader anchors on predictability before non-determinism. Loop validation (Ch 15) follows multi-agent (Ch 14) because validating a single loop is a prerequisite to trusting many. Self-evolution (Ch 16) sits before observability (Ch 17) as motivation — you can't safely evolve what you can't observe, so the reader feels the need before getting the tool.
- **The Design Debate sidebars are the neutrality mechanism.** Five recurring sidebars (mapped in "About this book") stage the real vendor disagreements at the chapters where they bite, presenting both camps rather than an answer. Given the author's Google affiliation, holding these scrupulously even-handed is what protects the book's vendor-neutral credibility — Google appears as one instantiation among five, never the default.
- **The vendor landscape (Ch 19) is one chapter, not a part — deliberately.** Instead of a chapter per platform (which bloats the page count and dates fastest), the convergence table is the single reference model and each vendor is a one-page instantiation. This is the main lever for hitting the ~500-page target while staying current.
- Keep case-study *and vendor* specifics (versions, metrics, product names) dated and quarantined in Part IV and the appendices — branding shifts fast (e.g., Vertex AI Agent Builder → Gemini Enterprise Agent Platform within a single year), so the pattern chapters in Parts I–III must stay product-free to age well.
- **Voice consistency is a first-class risk, like the Aegis thread.** Draft the *voiced* sections (openers, sidebars, closes) against the Voice & Style Guide, and resist letting the voice creep into the mechanics. If a chapter opener doesn't have a real scar or scene behind it, write it plainer rather than manufacturing an anecdote — a forced war story reads worse than none.

### Sources underpinning the examples
- InfoWorld, *Best practices for building agentic systems* (Apr 2026).
- Spotify Engineering: *Multi-Agent Architecture for Smarter Advertising* (Feb 2026); *Spotify x Anthropic Live* + Honk series (Nov 2025–Apr 2026); LLM evals (May 2026).
- IBM Think 2026 recap; A2A protocol v1.0; 2026 MCP roadmap.
- Vendor landscape (Ch 19): AWS Bedrock AgentCore docs & AWS ML blog; Anthropic *Building Effective Agents* (Dec 2024) and *Effective Context Engineering for AI Agents* (Sept 2025); Microsoft *Agent Framework 1.0* (GA Apr 2026) & Azure Architecture Center; Google Cloud Next 2026 / Gemini Enterprise Agent Platform docs; OpenAI *AgentKit* & Agents SDK; Andrew Ng (AI Ascent 2024); Cognition "Don't Build Multi-Agents"; Liu et al., *Agent Design Pattern Catalogue* (JSS 2025). *(Full detail in the companion vendor-landscape briefing.)*
