# Chapter 10: Composing an Agent


## The org chart

Picture what you are actually trying to build. Not one agent answering one question, but a system: a fleet of them, each good at one thing, handing work to each other. A router reads the incoming request and decides where it goes. A researcher gathers what is needed. A specialist does the hard part. A reviewer checks the work before it ships. It looks like an org chart, with individual agents composed into something that behaves like a competent team.

Try to resist the temptation to jump straight into the multi-agent system. The moment a multi-agent system looks appealing is usually the moment before you have made a single agent reliable, and a fleet of unreliable agents is a committee that loses your data. Earn the org chart by building one loop first. Build one loop that holds and you can compose the org chart. Skip that step and the org chart is only a more expensive way to fail.

## The heartbeat

So, one agent. What turns the pile of parts from Part II, a model, a context window, a memory store, a tool, a gate, into an agent at all is a single structural idea: the loop. The last gate is what separate an application from an agent. In application you ask it to do one thing, it execute and either make it or fail. The agent on the other hand reasons about what to do, does it, looks at what happened, and decides whether it is finished or goes around again. Reason, act, observe, repeat. That cycle is the heartbeat of an agent, you have a system that pursues a goal. It can take a step, see that the step failed, and try a different one, all without you in between. That difference, the model deciding the next move at runtime instead of following a script you wrote, is the entire reason agents are worth the trouble.

This shape is settled. Modern coding agents run it at industrial scale, a long "while not done" loop that plans, edits, runs the tests, reads the errors, and tries again for an hour without a human. The shape barely changes. The work is in engineering it well, which is what this part is about.

## The anatomy of a well-built loop

A loop that survives contact with production has five parts, and a loop that fails in production is usually missing one of them.

A **goal** requires a reliable definition of done. While a deterministic, machine-checkable signal (like a compiler) is ideal, many real-world tasks are inherently semantic and require a model to evaluate success. To prevent a model from hallucinating victory, you cannot rely on the generation step's vague intuition. Instead, isolate evaluation as a distinct step in your execution lifecycle through an explicit multi-turn state machine. You do this by dynamically mutating the system context between turns—explicitly stripping the agent's "creative" instructions and replacing them with a strict "critical" evaluation rubric. The evaluation step must map these semantic criteria onto a strict, machine-readable schema (such as a boolean completion flag).

The **context** is what the agent sees each turn. Every trip around the loop, the agent needs a clear view of the world, but in a multi-turn state machine, history quickly accumulates noisy garbage. If left unchecked, the loop drifts, forgets the original ask, or fixates on its own past mistakes. To maintain context integrity, the runtime must employ active memory management. Instead of passing raw, bloating conversation histories, the application must use context distillation (summarizing past execution states) or semantic choice (filtering the history to pass only the original prompt, the latest critique, and the current revision delta).

The **actions** the agent can take, and the feedback it receives, include the actual outputs of the tools called. The system must maintain a strict division of labor regarding errors. The agent should only see and reason about functional or semantic errors (e.g., "Database returned 0 rows, try a synonym" or "Validation failed for field X"), treating them as routine observations to pivot against. Systemic, deterministic infrastructure errors (e.g., 429 Rate Limits, network timeouts, or expired auth tokens) should be completely abstracted away from the agent's view. A loop that asks an AI to reason about a dropped TCP connection is a loop that wastes tokens.

A **termination** policy provides the ultimate boundary. The entire loop must be bound by deterministic infrastructure guardrails like a maximum iteration cap or cost ceiling. Because the agent cannot be trusted to terminate on its own, the application layer must own the final lifecycle state. When a loop hits its hard iteration cap without a successful evaluation flag, the application must execute a deterministic fallback strategy: it either surfaces the last evaluated "best-effort" draft alongside an engineering warning flag, reverts to a safe baseline state, or gracefully escalates the state to a human-in-the-loop review queue.

An **error strategy** dictates what the loop structure does when a step fails, retries, or gives up. Failure is a routine event that the architecture must handle as a matter of course. At the network level, the application infrastructure handles automated retries with exponential backoff. At the cognitive level, if the agent's evaluation state flags a failure multiple turns in a row (indicating an execution deadlock or "ping-ponging"), the error strategy triggers a prompt-level intervention—such as injecting a "circuit-breaking" hint that forces the agent to try an entirely different logical path rather than making micro-tweaks to a broken approach.

## The Loop Flow

You will build the same few shapes over and over. Naming them helps you reach for the right one.

**The ReAct Flow (Action-Oriented) **

This is the foundational pattern where the agent leans heavily on tool use to explore an unpredictable environment. The cycle is a tight, reactive cadence: _Thought → Action → Observation._

The model uses its reasoning step to figure out which tool to call, executes it, and uses the tool's output to determine its next move. It is highly adaptive, making it the perfect starting point for open-ended discovery tasks like searching databases or exploring local code bases. However, because it evaluates its state completely from scratch on every single turn, it gives the model maximum freedom—meaning it has the highest risk of drifting or getting caught in an erratic, non-deterministic loop if your prompt instructions aren't tightly bounded.

**The Temporal Reflection Flow (Quality-Oriented)**

This modality focuses entirely on iterative refinement and output quality rather than tool execution. Because we are using a single agent, you avoid self-grading confirmation bias by executing a **temporal state split** across turns.

Instead of letting the agent write and evaluate freely, your application code forces the agent through a strict multi-turn sequence. On Turn 1, the system context commands the model to create a draft. On Turn 2, your application actively mutates the system context, explicitly stripping the "creative" instructions and replacing them with a strict, critical evaluation rubric: "Review your previous draft. Check for accuracy and tone, and output your critique." By shifting the model’s cognitive focus across time using your application code as the state machine, a single agent can objectively optimize its own work until it hits your definition of done.

**The Plan-Then-Execute Flow (Efficiency-Oriented)**

This modality separates the macro-thinking from the micro-doing to protect your token budget and reduce latency. Instead of reasoning from scratch at every single turn, the single agent uses its first turn to draw up an entire structured execution plan (an array of sub-tasks).

The infrastructure then loops through the plan, executing the steps sequentially. The agent only steps back into a "reasoning" state if an execution step returns a semantic surprise or a validation failure, forcing a mid-course correction. This pattern gives you a highly predictable trajectory that you can inspect or gate before any code runs. The trade-off is rigidity; if the model makes a bad assumption during the initial planning phase and your error boundaries don't catch the drift early, the loop will confidently execute a flawed roadmap to the end.



## TODO: running example

> **TODO (author):** the running-example section goes here, to be written once the running example is settled.
>
> *What this section should do:* Introduce the top-level loop the running example runs on every trigger, the reason-act-observe cycle it repeats while working a task. Show the five parts concretely: the verifiable goal (the alerting condition clears), the per-turn context, the tools and their honestly-fed-back results, the termination policy, and the error strategy. Make termination vivid, an incident agent stuck retrying a failing remediation is exactly the runaway loop that eats an API budget at 3 a.m., so show the iteration cap and the no-progress detector all wired in. Name the loop shape it uses (a ReAct loop with a human gate on remediation), and set up how the rest of Part III will deepen it.

## Beyond the Single Loop
Once your single-agent loop is rugged, actively managing its context, respecting its boundaries, and handling its own errors, you have a reliable, bounded primitive. You have successfully built a single, competent worker.

But a single worker has a cognitive ceiling. If you force a lone agent loop to handle an entire end-to-end software feature or a massive, multi-step business process, the architecture breaks down. The context window fills with history noise, tool selection degrades, and the prompt collapses under the weight of too many conflicting instructions. To tackle complex, long-horizon problems, you cannot just make the loop longer. You have to break the work apart.

This is the exact boundary where we move from a single loop to a workflow, and from a lone actor to a multi-agent system.

By taking your rugged single loop and treating it as an isolated node in a larger execution graph, you can begin to compose the org chart we talked about at the beginning. You can chain specialized loops together. One node runs a loop dedicated entirely to unstructured research, its validated JSON output boots up a second node running a loop focused strictly on code synthesis, and that routes to a third loop optimized solely for defensive testing and critique.

Now that you have built a single loop that does not break, you have earned the right to build the fleet. In the next chapter, we will look at how to design these cyclic graphs, establish clean data contracts between specialized agents, and orchestrate them into a cohesive, production-grade workflow.


