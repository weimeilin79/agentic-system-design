# Chapter 11: Workflow Patterns and Cyclic Graph Shapes

---

When you move past a single agent loop, you enter the world of workflows. In production, a workflow is structured as a graph. Your agents, prompts, and deterministic code routines become nodes. The execution paths between them become edges.

The moment you introduce loops into these workflows, your graph becomes cyclic.

Managing a cyclic graph in standard software engineering is already a challenge. Managing one where the nodes behave probabilistically is dangerous. If you let nodes connect to each other without strict constraints, your application will quickly degenerate into an unreviewable tangle of infinite loops, runaway token costs, and untraceable state drift.

To build a production-grade workflow, you must rely on a few disciplined, proven architectural shapes. You do not design a custom web of chaos. Instead, you map your problem to one of five foundational cyclic topologies.

## The Spectrum of Graph Control
Before picking a shape, you must understand the trade-off between structural predictability and agent autonomy.

- **Linear-Cyclic:** High predictability, low autonomy.
- **Central Hub:** Managed routing, balanced autonomy.
- **Parallel-Cyclic:** Isolated concurrency, localized autonomy.
- **State-Chart Mesh:** Fluid transitions, high autonomy.
- **Hierarchical Rings:** Scoped complexity, deep autonomy.

As you move down this list, the system becomes more flexible and capable of handling open-ended problems. It also becomes significantly harder to test, monitor, and debug. Always default to the most rigid shape that can solve your problem. Reach for fluid autonomy only when a predictable pipeline hits a hard wall.

## The Law of Controlled Routing
Every cyclic shape, no matter how autonomous, obeys a single architectural law: **The model may choose the route, but the infrastructure defines the map.**

In some systems, your edges are completely static: your application code evaluates a state variable and forces a transition. In other systems, you use dynamic routing, allowing an LLM node to emit a decision string that directly dictates the next node execution.

Dynamic routing does not mean lawless routing. If a model controls the steering wheel, your application code must still enforce the guardrails:

- **Literal Enum Boundaries:** The model must choose its destination from a strict, code-defined set of strings or enums via structured outputs. It cannot invent a destination node.
- **Deterministic Invariants:** Your infrastructure can always intercept a routing choice. If the model chooses a destructive action node but the application detects that the user lacks proper permissions, the code overrides the model and forces a redirection.
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
Execution moves deterministically from Node A to B to C. Each node builds on top of the payload of the previous node, optimizing for a single, narrow objective. Node D acts as the quality assurance boundary. It evaluates the compiled final output against the global definition of done. If the output fails validation, Node D injects a critique payload and routes execution all the way back to an earlier stage to start a refinement pass.

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
The Central Hub inspects the current state of the workflow and decides which node to trigger next. Once Node A finishes running its tool or loop, it has no awareness that Node B exists. It simply outputs its structured data payload to the Hub. The Hub updates the global state object, evaluates whether the overall goal is met, and either routes to the next leaf node or exits the workflow.

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
Node A splits a large workload into independent pieces, such as searching for financial data across three different fiscal years concurrently. Nodes B1, B2, and B3 run in parallel. If B1 hits an execution error or misses a key field, it runs an internal micro-cycle to heal itself or re-query without blocking the other branches. Once all parallel branches exit their local loops cleanly, Node C collects the structured results and merges them.

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
Instead of returning to a central hub, Node A finishes its task, evaluates the immediate result, and triggers Node B directly. If Node B hits a specific condition, it might route directly to Node C, or loop right back to Node A. The state object travels along these paths like a token in a network.

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
The outer macro-loop manages the high-level roadmap, like breaking a massive software architecture specification down into individual microservices. It passes one microservice specification down to the inner micro-loop. The inner loop spins aggressively by writing code, running a compiler, fixing syntax errors, and refactoring until the individual unit passes its local tests. Once the inner loop breaks cleanly, control returns to the macro-loop to evaluate the next high-level step.

### When to reach for it
This is the standard topology for advanced autonomous engineering agents or automated legal contract drafting suites. It allows the system to focus intensely on polishing a tiny detail without losing track of the massive end-to-end objective.
