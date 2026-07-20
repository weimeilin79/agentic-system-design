# Chapter 12: Workflow Patterns and Cyclic Graph Shapes

---

Once you establish your state boundaries and choose your delegation rungs, you must lay down the concrete geometric tracks for your architecture. In production, a complex workflow is structured as an execution graph. The system operates as a high-level **macro-loop** that coordinates data movement across the entire topology.

Within this macro-loop, your graph nodes are entirely polymorphic. A single node is no longer just a raw text prompt. It can be a specialized stateful agent running its own micro-loop, a stateless deterministic tool executing an API call, or a human operator serving as a manual validation gate.

Plaintext

```
===================================================================================
THE MACRO-LOOP WORKFLOW
===================================================================================
               ┌──────────────────────────────────────────────────┐
               ▼                                                  │ (Macro Back-Edge)
   ┌──────────────────────┐            ┌──────────────────────┐   │
   │      NODE A:         │ ────────►  │      NODE B:         │ ──┴
   │   Stateful Agent     │            │   Stateless Tool     │
   │  (Runs Micro-Loop)   │            │   (Executes API)     │
   └──────────────────────┘            └──────────┬───────────┘
                                                  │
                                                  ▼
                                       ┌──────────────────────┐
                                       │      NODE C:         │
                                       │   Human Checkpoint   │ ──► [ Exit Graph ]
                                       │   (Manual Gateway)   │
                                       └──────────────────────┘
```

The moment you introduce loops into these workflows to allow refinement, error recovery, or interactive feedback, your graph becomes cyclic. Managing a cyclic graph where nodes behave probabilistically is highly dangerous. If you let nodes connect to each other without strict constraints, your application will quickly degenerate into an unreviewable tangle of infinite loops, runaway token costs, and untraceable state drift. To build a production-grade workflow, you must map your system to one of five foundational cyclic topologies.

## The Spectrum of Graph Control
Before picking a shape, you must understand the trade-off between structural predictability and system autonomy.

- **Linear-Cyclic:** High predictability, low autonomy.
- **Star-Cyclic:** Managed routing, balanced autonomy.
- **Parallel-Cyclic:** Isolated concurrency, localized autonomy.
- **Arbitrary Cyclic Network:** Fluid transitions, high autonomy.
- **Nested-Cyclic:** Scoped complexity, deep autonomy.

As you move down this list, the system becomes more flexible and capable of handling open-ended problems. It also becomes significantly harder to test, monitor, and debug. Always default to the most rigid shape that can solve your problem. Reach for fluid autonomy only when a predictable pipeline hits a hard wall.

## The Law of Controlled Routing
Every cyclic shape, no matter how autonomous, obeys a single architectural law: **The model may choose the route, but the infrastructure defines the map.**

In some systems, your edges are completely static: your application code evaluates a state variable and forces a transition. In other systems, you use dynamic routing, allowing an LLM agent node to emit a decision string that directly dictates the next node execution. If a model controls the steering wheel, your application code must still enforce the guardrails:

- **Literal Enum Boundaries:** The model must choose its destination from a strict, code-defined set of strings or enums via structured outputs. It cannot invent a destination node.
- **Deterministic Invariants:** Your infrastructure can always intercept a routing choice. If an agent chooses a destructive action node but the application detects that the user lacks proper permissions, the code overrides the model and forces a redirection.
- **The Dead-End Fallback:** If a model returns an invalid route or enters an evaluation deadlock, the runtime strips its routing authority and defaults to a hardcoded safe node, such as a human review queue.

## 1. The Multi-Stage Pipeline with Global Back-Edge (Linear-Cyclic)
This shape mimics a traditional compiler or a manufacturing assembly line. Data flows strictly forward through specialized, sequential execution stages, but it hits a strict gatekeeper at the very end.

Plaintext

```
                                (Fails Global Validation Pass)
       ┌────────────────────────────────────────────────────────────────────────┐
       │                                                                        │
       ▼                                                                        │
┌──────────────┐       ┌──────────────┐       ┌──────────────┐       ┌──────────┴───┐
│   Node A     │ ──►   │   Node B     │ ──►   │   Node C     │ ──►   │    Node D    │
│ (Ingestion)  │       │ (Extraction) │       │ (Synthesis)  │       │ (Gatekeeper) │
└──────────────┘       └──────────────┘       └──────────────┘       └──────┬───────┘
                                                                            │
                                                                            ▼ (Pass)
                                                                       [ Exit Graph ]
```

### Mechanics
Execution moves deterministically from Node A to B to C. Each node builds on top of the payload of the previous step. Node D acts as the quality assurance boundary. This role can be populated by a hostile evaluation agent, a deterministic code scanner, or an asynchronous human validation interface. If the output fails validation, Node D injects a critique payload and routes execution all the way back to an earlier stage to start a refinement pass.

### Defensive Design: Structured Criticism & Edge Budgets

- **The Trap:** If Node D passes a loose, free-text critique back to Node A, the upstream agent nodes will enter a semantic deadlock, making superficial edits or hallucinating compliance across endless loops.
- **The Pattern:** Force Node D to output a strict, schema-validated checklist of missed criteria or an explicit code diff. Additionally, the back-edge must maintain a monotonic macro-counter. If execution traverses this global back-edge more than three times, the application layer triggers a circuit breaker, terminates the graph, and alerts an engineer.

### When to reach for it
This is the standard pattern for text synthesis, report generation, and automated code creation. It allows individual steps to focus entirely on forward progress without managing global workflow state, while guaranteeing quality at the final exit boundary.

## 2. The Central Hub Router (Star-Cyclic)
In this shape, state is managed centrally by a supervisor or a deterministic routing node. Leaf nodes execute highly specific sub-tasks, are completely decoupled from each other, and always return their results directly back to the central hub.

Plaintext

```
          ┌──────────────────────────────────────┐
          │         NODE 0: Central Hub          │◄────────────────┐
          │       (Router / State Manager)       │                 │
          └────┬──────────────┬──────────────┬───┘                 │
               │              │              │                     │
               ▼              ▼              ▼                     │
         ┌──────────┐   ┌──────────┐   ┌──────────┐                │
         │  Node A  │   │  Node B  │   │  Node C  │                │
         │ (Search) │   │ (Verify) │   │ (Write)  │                │
         └─────┬────┘   └─────┬────┘   └─────┬────┘                │
               │              │              │                     │
               └──────────────┴──────────────┴─────────────────────┘
                             (Return Payload to Hub)
```

### Mechanics
The Central Hub acts as the macro-loop director. It inspects the current state of the workflow and decides which node to trigger next. The target node can be a simple data-fetching tool, a specialized worker agent, or a human-in-the-loop approval gate. Once Node A finishes running, it has no awareness that Node B exists. It simply outputs its structured data payload back to the Hub. The Hub updates the global state object, evaluates whether the overall goal is met, and routes accordingly.

### Defensive Design: Isolated Memory

- **The Trap:** If a single model instance is acting as the hub and iteratively calling leaf nodes, its context window will quickly bloat with the logs, formatting headers, and execution histories of every node it has touched, degrading its routing accuracy.
- **The Pattern:** Treat leaf nodes as completely decoupled execution sandboxes. The Hub should append only the final, structured JSON data payload returned by the leaf node or human response to its global state object, completely wiping the leaf node's raw internal conversational scratchpad before the next turn begins.

### When to reach for it
Reach for this pattern when building complex, feature-rich systems where capabilities need to be toggled on or off dynamically. It is easy to audit, test, and expand. If you need to add a new capability, you build Node D and update the router logic in the Hub. The existing leaf nodes remain untouched.

## 3. Fork-Join with Local Recovery Loops (Parallel-Cyclic)
When a massive task can be split into concurrent chunks, you map the sub-tasks out to parallel branches. To prevent a single downstream error from failing the entire application, each branch contains its own tight cyclic recovery loop.

Plaintext

```
                                ┌───► [ Node B1: Search ] ──┐
                                │          ▲        │       │
                                │          └ (Retry)┘       ▼
┌──────────────────┐            │                    ┌─────────────┐
│  Node A: Fork    │ ───────────┼───► [ Node B2: Search ] ──►│ Node C: Join│
│ (Split Context)  │            │          ▲        │       │ (Merge &    │
└──────────────────┘            │          └ (Retry)┘       ▼  Validate)  │
                                │                           │             │
                                └───► [ Node B3: Search ] ──┘             └───┬─────────┘
                                           ▲        │                         │
                                           └ (Retry)┘                         ▼
                                                                        [ Exit Graph ]
```

### Mechanics
Node A splits a large workload into independent pieces, such as searching for financial data across three different fiscal years concurrently. Nodes B1, B2, and B3 run in parallel. Each node can be an autonomous worker agent or a tool execution harness. If B1 hits an execution error or misses a key field, it runs an internal micro-cycle to heal itself or re-query without blocking the other branches. Once all parallel branches exit cleanly, Node C collects the results and merges them.

### Defensive Design: Hydrated Clones & Branch Timeouts

- **The Trap:** If Node B2 experiences a continuous API failure or gets caught in an internal data-healing loop, it can completely hang the global application thread, leaving the merged state in Node C in a permanent deadlock.
- **The Pattern:** Never let parallel branches mutate a shared global state object directly. Instead, instantiate each branch with a deeply cloned, isolated snapshot of the context. Furthermore, enforce an asynchronous branch-level timeout, such as 30 seconds. If B2 fails to resolve its local loop within that window, its execution context is cancelled, and Node C is forced to run an alternative merge strategy or route to a human exception handler using a partial success state flag.

### When to reach for it
This shape maximizes throughput and lowers latency for heavy data processing workloads. It isolates localized data failures so they do not cascade up to the main application thread.

## 4. The State-Chart Mesh (Arbitrary Cyclic Network)
This shape mimics a classic finite state machine where almost any node can transition to any other node based on the evolving state payload. There is no central supervisor routing traffic. The routing logic is distributed across the transition edges themselves.

Plaintext

```
      ┌───────────────┐               ┌───────────────┐
  ───►│    Node A     │ ◄───────────► │    Node B     │
      │ (Diagnostics) │               │   (Repair)    │
      └───────┬───────┘               └───────┬───────┘
              ▲                               │
              │                               ▼
              │                       ┌───────────────┐
              └───────────────────────┤    Node C     │
                                      │ (Verification)│
                                      └───────────────┘
```

### Mechanics
Instead of returning to a central hub, Node A finishes its task, evaluates the immediate result, and triggers Node B directly. The network token passes dynamically from node to node. Node A might be a diagnostic agent, Node B an infrastructure repair tool, and Node C a human authorization checkpoint. If Node C rejects the repair, it can route control back to Node A to start diagnostics over again.

### Defensive Design: Monotonic Token Energy

- **The Trap:** Because routing is decentralized across multiple autonomous agents and nodes, tracking global iteration limits becomes incredibly difficult. Nodes might pass execution back and forth across dozens of iterations without ever triggering a single node-level constraint.
- **The Pattern:** Embed a `graph_energy` property directly into the traveling global state payload. Every single edge transition, regardless of whether it is triggered by an agent, tool, or human step, decrements this integer by 1. The moment the `graph_energy` token reaches zero, the current edge transition is instantly short-circuited by the infrastructure, forcing an unconditional redirection to a human troubleshooting queue.

### When to reach for it
This is the shape you reach for when building complex, open-ended diagnostic systems, like an automated network troubleshooting system. The system needs to pivot dynamically from gathering data, to applying a fix, to verifying, and potentially back to a completely different type of data gathering depending on what the fix uncovered.

## 5. Hierarchical Nested Rings (Nested-Cyclic)
This shape handles massive, multi-layered tasks by nesting micro-loops inside a larger macro-loop. It is a structure of rings running inside other rings.

Plaintext

```
┌────────────────────────────────────────────────────────┐
│  MACRO LOOP: Epic / Feature Generation Pipeline       │
│                                                        │
│  ┌──────────────────────┐      ┌────────────────────┐  │
│  │ NODE A: Plan / Spec  │ ───► │ NODE B: Build Unit │  │
│  └──────────────────────┘      └─────────┬──────────┘  │
│              ▲                           │             │
│              │                           ▼             │
│              │                 🌀 MICRO LOOP:          │
│              │                    Code Synthesis       │
│              │                    & Compilation        │
│              │                           │             │
│              └───────────────────────────┴─────────────┤
│                     (Unit Fails Architectural Gate)    │
└────────────────────────────────────────────────────────┘
```

### Mechanics
The outer macro-loop manages the high-level roadmap, like breaking a massive software architecture specification down into individual microservices. It passes one microservice specification down to an inner micro-loop. The inner loop spins aggressively by writing code, running a compiler, fixing syntax errors, and refactoring until the individual unit passes its local tests. Once the inner loop breaks cleanly, control returns to the macro-loop to evaluate the next high-level step.

### Defensive Design: Ephemeral Scratchpads & Compressed Commits

- **The Trap:** If an inner worker agent takes 15 compilation turns to debug a function, allowing that massive conversational trial-and-error log to leak back up into the macro-loop will instantly trigger massive history inflation, blowing your token budget and blinding the macro planner agent.
- **The Pattern:** Run the inner micro-loop inside an entirely fresh, throwaway context window, an ephemeral scratchpad. When the inner loop achieves its definition of done, it purges its internal reasoning history entirely. It returns only a single, heavily compressed commitment payload, such as the final verified source code and a boolean test status, back up to the macro state machine.

### When to reach for it
This is the standard topology for advanced autonomous engineering agents or automated legal contract drafting suites. It allows the system to focus intensely on polishing a tiny detail without losing track of the massive end-to-end objective.

## The Architecture of Wheels Within Wheels
Ultimately, building a production-grade cyclic graph means structuring your architecture as a hierarchical wheel of wheels. At the macro-level, your host application code remains the deterministic skeleton. It acts as a rigid state machine that manages the global execution graph, governing exactly how data moves between specialized agent nodes like your planner, executor, and evaluator. When a downstream gatekeeper flags a systemic failure or requirement mismatch, the infrastructure intercepts the global state and fires a macro-back edge. This routes the execution backward across the system graph to trigger strategic replanning, ensuring you do not let a localized component blindly patch over a foundational structural flaw.

Beneath this global choreography lies the micro-topology, where non-deterministic model autonomy is intentionally trapped inside isolated containment zones. Each agent node functions like a black-box microservice wrapped in strict data contracts, running its own localized internal loop tailored to its specific task. Your planner can spin in a tight reflection loop to output validated JSON schemas, while your executor runs a focused ReAct cycle to safely explore an erratic tool landscape. This clean separation guarantees that the chaotic trial and error of model reasoning cannot leak out to corrupt your broader application state. By engineering your graphs with this boundary, you can ruthlessly optimize individual agent behaviors without ever destabilizing the overarching system rails.
