# Chapter 12: Multi-Agent Orchestration

Multi-agent orchestration is fundamentally an optimization strategy for context windows and compute efficiency. When you scale a single-agent loop to handle complex, long-horizon tasks, you run into a hard architectural ceiling. Packing dozens of tool definitions, tracking variable schemas, and accumulating a massive log of trial-and-error observations inside a single context window triggers severe attention degradation. As the history balloons, the model's tool selection precision drops, it begins to misinterpret complex instructions, and the cost of processing identical system prompts over long sequences scales exponentially.

To bypass this ceiling, production engineering treats multi-agent design as a distributed computing problem. Instead of expanding a lone agent's cognitive load, the system splits the workflow across a network of isolated, highly specialized execution nodes. The winning industry paradigm is a centralized orchestrator that owns the global state and spawns transient, single-purpose subagents to execute specific sub-tasks. These subagents operate within a strictly bounded runtime, pass a compressed data payload back to the orchestrator, and immediately evaporate. This layout isolates non-deterministic reasoning loops, prevents history bleed, and keeps processing costs entirely predictable.

## When You Actually Need More Than One Agent

You should only decompose your system into multiple agents if you meet at least one of these four production criteria:

- **Context Caps: **The data required to solve a problem exceeds a single model's attention window. Subagents are deployed to burn independent context budgets on specific document chunks, returning a highly compressed summary. This is context management wearing an agent costume.
- **Security Isolation: **Compliance or security dictates that an agent handling untrusted external inputs must never touch internal infrastructure, database tools, or financial APIs. Splitting the architecture becomes an explicit security control where isolation is the entire point.
- **Hard Parallelism:** You must execute multiple data-gathering pipelines concurrently with zero dependencies, dropping wall-clock latency from forty seconds to four. If parallel branches must sync or check in with each other mid-flight, the parallelism is a mirage.
- **Model Heterogeneity: **Burning tokens on an expensive, ultra-slow frontier reasoning model for basic text extraction or regex parsing is a massive infrastructure failure. Splitting lets you assign low-cost micro-models to routine tasks and reserve premium engines for deep reasoning checkpoints.

If your requirements do not hit one of these four pillars, build a single agent. Fix its tools, refine its context schemas, and tighten its loop. Every operational problem you hope to solve by adding an agent is more reliably so

## Agents, Tools, and Who Holds the State

The question underneath "do I need another agent?" is almost always "should this capability be a tool or an agent?" The definitive line between them is **state**.

A tool is stateless. When your framework calls `metrics_query(service, window)`, the tool executes, returns a raw data payload, and retains nothing. Every bit of persistence—what has been tried, what failed, and what was learned—lives entirely in the caller's memory window.

An agent is stateful. It accumulates a running history, updates an internal memory store, executes its own micro-evaluation loop, and maintains a changing picture of its environment.

Promoting a tool to an agent is not a routine decision about scaling capabilities. It creates a second repository of truth. In distributed systems, creating a second unmanaged state store is exactly how you create a split-brain crisis—like two conflicting accounting legers reporting completely different definitions of revenue. A tool cannot hold a divergent truth because it holds nothing; an agent can.

```
┌─────────────────────────────────────────────────────────────────────────┐
│ THE THREE RUNGS OF DELEGATION                                           │
├─────────────────────────────────────────────────────────────────────────┤
│ Rung 1: Plain Tool      │ Stateless call. Leaves no footprint behind.   │
├─────────────────────────┼───────────────────────────────────────────────┤
│ Rung 2: Agent-as-a-Tool │ Stateful internal loop. Stateless return.     │
├─────────────────────────┼───────────────────────────────────────────────┤
│ Rung 3: Delegated Agent │ Stateful lifecycle. Two-way parent sync.      │
└─────────────────────────────────────────────────────────────────────────┘
```

This structural risk gives you your architectural default: **prefer a tool**. You should only climb the three rungs of delegation when the complexity of the workload forces your hand:

1. The **Plain Tool**: A stateless, deterministic function call.
1. The **Agent-as-a-Tool**: A stateless call with an autonomous loop trapped inside it. It accepts arguments, reasons across several internal turns to resolve an ambiguity, returns a clean result, and completely wipes its scratchpad memory. From the orchestrator's perspective, it is a black-box tool that belongs in a standard tool-definition list rather than a complex graph topology. This is the exact pattern frameworks converged on when they transitioned from loose supervisor libraries to supervisor-as-a-tool architectures.
1. The **Delegated Agent**: The only rung that represents an independent task rather than an encapsulated call. It possesses a distinct lifecycle, can communicate asynchronously with its parent while it works, and carries its own context across long timelines. Reach for this rung only when a sub-task genuinely needs to remember its own operational history across separate invocations.

## The Immutable Rule: One Canonical Record

When your architecture forces you to climb to Rung 3, you should have one copy of the state, with the full history. Each delegated agent will inevitably maintain its own chat transcript while its internal loop runs, and that is acceptable. What matters is that the host infrastructure maintains a single, canonical record of the entire system: what is known, what was delegated, what returned, and in what exact sequence.

```
                       ┌─────────────────────────┐
                       │  Canonical State Store  │
                       │  (System Event History) │
                       └────────────┬────────────┘
                                    │
           ┌────────────────────────┴────────────────────────┐
           ▼                                                 ▼
┌──────────────────────┐                          ┌──────────────────────┐
│   Agent A Segment    │                          │   Agent B Segment    │
│  Writes to: key.aa   │                          │  Writes to: key.ab   │
└──────────────────────┘                          └──────────────────────┘
```

Agents can read from this central registry and write their results back to keys they explicitly own. This design eliminates the primary failure mode of multi-agent networks. A long-lived, private copy of a shared fact that one agent edits on the side and attempts to reconcile with the parent later.

This rule is visible in how modern production frameworks are built:

- Google's ADK: An ADK session is simply an ordered event history of the system combined with a session.state dictionary of working facts. The parent hands a scoped invocation context down to sub-agents. Every component reads and writes to that single session, while routing results into named keys where key prefixes scope their visibility and lifetime.
- LangGraph: This architecture enforces a strongly typed state object that flows through the graph. A reducer function cleanly merges each node’s update, and a checkpointer persists the delta.

## Protocol Separation: MCP vs. A2A

Your delegation choices dictate your network protocols. The industry has separated these communication channels along the exact same lines as the rungs of state:

- Model Context Protocol (MCP): This is how an agent interacts with tools. Its unit of exchange is a simple request-response call that returns a payload and terminates. Both a plain tool and an encapsulated agent-as-a-tool communicate over MCP.
- Agent-to-Agent (A2A) Protocol: This is how an agent communicates with an independent, stateful peer. Its unit of exchange is a long-lived task with an active lifecycle, because the receiving end must maintain its own state and may take minutes or hours to compile a resolution.

If you find yourself trying to implement an A2A protocol for an operation that could be fulfilled by a standard function return, your architecture is sitting on the wrong rung. 

## The Cost of the Split
Multi-agent architecture is not a structural badge of honor. It is an engineering tax you pay when a single context window can no longer sustain the cognitive load of your workload. Every new agent boundary you draw across your system introduces a fresh border crossing. That crossing requires strict data customs, explicit contract validation, and unavoidable network latency.

If you manage these borders loosely, your runtime will rapidly fracture into a chaotic web of split-brain state conflicts and drifting realities. If you manage them with rigorous infrastructure discipline, anchoring your application to a single canonical ledger and treating auxiliary loops as encapsulated tools, you transform erratic model reasoning into a predictable distributed system.

### The Architectural Default
Keep your state unified. Keep your subagent lifecycles short. Never let a model own the database of truth. Protect your token margins and context windows ruthlessly, and only split your models when the work leaves you no other choice.

Once you have drawn these explicit data boundaries and established exactly who owns the state, you are ready to move from the philosophy of the border to the layout of the track. In Chapter 11, we will map out the concrete geometric topologies and cyclic graph shapes required to execute these multi-agent workflows safely in production.