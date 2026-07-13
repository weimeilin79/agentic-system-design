# Chapter 10: Composing Agents

*Draft in the author's voice. Opens Part III (Composition). Framing chapter, more prose than catalog: opens on the multi-agent vision, pivots to the single loop as the building block, then defines the loop, its shapes, and its termination controls, and sets up the rest of the part. Running-example section left as a TODO block. No em-dashes; a few ellipses. Which phase: the loop.*

---

## The org chart

Picture what you are actually trying to build. Not one agent answering one question, but a system: a fleet of them, each good at one thing, handing work to each other. A router reads the incoming request and decides where it goes. A researcher gathers what is needed. A specialist does the hard part. A reviewer checks the work before it ships. It looks like an org chart, and the org chart is the goal: individual agents composed into something that behaves like a competent team.

Resist building it yet. There is a trap in that picture, and the field has walked into it enough times to name it: the moment a multi-agent system looks appealing is usually the moment before you have made a single agent reliable, and a fleet of unreliable agents is a committee that loses your data. So earn the org chart. Every box in it, the router, the researcher, the reviewer, is the same thing underneath: a loop. Build one loop that holds and you can build a hundred. Skip that step and the org chart is only a more expensive way to fail.

## The heartbeat

So, one agent. What turns the pile of parts from Part II, a model, a context window, a memory store, a tool, a gate, into an agent at all is a single structural idea: the loop.

Here is the whole thing in one line. The agent reasons about what to do, does it, looks at what happened, and decides whether it is finished or goes around again. Reason, act, observe, repeat. That cycle is the heartbeat, and once it is beating you no longer have a text generator that answers a question, you have a system that pursues a goal. It can take a step, see that the step failed, and try a different one, all without you in between. That difference, the model deciding the next move at runtime instead of following a script you wrote, is the entire reason agents are worth the trouble.

This shape is settled. Modern coding agents run it at industrial scale, a long "while not done" loop that plans, edits, runs the tests, reads the errors, and tries again for an hour without a human. The shape barely changes. The work is in engineering it well, which is what this part is about.

## A loop is not a chain

The most useful distinction in this whole part is small and easy to miss, so I will make it loudly. A chain is linear and fixed: step A, then B, then C, in that order, every time. A loop is cyclic and revisable: it can repeat a step, branch, change course, or stop, based on what it just observed. A chain runs once through a sequence you decided in advance. A loop decides, at runtime, what comes next.

That single property, who chooses the next step, is the fork the rest of Part III lives on. When the steps are known ahead of time and never need to change, you want a chain, because it is predictable, auditable, and cheap, and the next chapter is about pushing as much of your system as you can onto that side of the line. When the path cannot be known in advance, when real-world messiness means the agent has to look and decide, you want a loop, and you pay for that flexibility with everything that can go wrong when a non-deterministic thing gets to choose its own next move. Almost every real system is a blend: a planned skeleton with adaptive loops inside each joint. Knowing which parts deserve which is most of the craft.

## The anatomy of a well-built loop

A loop that survives contact with production has five parts, and a loop that fails in production is usually missing one of them.

A **goal** it can actually test. The loop needs to know when it is done, and "the model decides it is done" is not good enough, because a language model has no built-in sense of finished and will happily declare victory or grind forever. The goal needs a verifiable stopping condition: the tests pass, the record matches, a checker returns true. The closer your goal comes to a machine-checkable signal, the more reliable everything downstream becomes, which is why loops over code work so well, the compiler is an honest judge.

The **context** it sees each turn. Every trip around the loop, the agent gets a fresh look at the world, and what you choose to put in that view is the whole of Chapter 5 doing its work inside the cycle. Get it wrong and the loop drifts, forgets the original ask, or fixates on noise.

The **actions** it can take. The tools from Chapter 7, and critically, the results of those actions fed back in honestly, including the failures. The best loops treat an error as just another observation: the tool returned "permission denied," the agent reads that and reasons about what to do next. A loop that hides its errors is a loop that repeats them.

A **termination** policy. This is the part everyone skips and everyone regrets, so it gets its own section below.

An **error strategy**. What the loop does when a step fails, retries, backs off, tries another approach, or gives up and escalates. Failure is not the exception in a long-running loop, it is a routine event the loop has to handle as a matter of course.

## The loop shapes

You will build the same few shapes over and over. Naming them helps you reach for the right one.

**The ReAct Loop** is the foundation, the single agent that reasons, calls one tool, observes, and goes again until done. It is the most broadly useful and the best understood, and it is where you should start every time. Reach for something more elaborate only when a plain ReAct loop hits a wall you can name.

**The Plan-Then-Execute Loop** separates thinking from doing: the agent draws up the whole plan first, then executes the steps, reasoning again only when a step surprises it. This spends fewer model calls than reasoning from scratch at every turn, which matters at scale, and it gives you a plan you can inspect or gate before any action runs. The cost is rigidity, a plan made up front is a plan that can be wrong about a world it has not looked at yet.

**The Reflection Loop** adds a critic: the agent produces something, then evaluates its own work against the goal and revises, looping on quality rather than on task progress. This is where a lot of the gain in output quality lives, and it is the seed of the self-improving loop we will not reach until Chapter 15.

**The Orchestrated Loop** is a loop whose steps are themselves other agents, a coordinator running sub-agents, each with its own inner loop. This is the multi-agent shape, and it is powerful and expensive and easy to get wrong, which is why it gets its own chapter (Chapter 13) and a standing warning to avoid it until a single agent has genuinely failed you.

**The Human-Gated Loop** is any of the above with a person wired into the cycle at the points that matter, the pattern from Chapter 8, viewed now as a structural element of the loop rather than a safety feature bolted on.

## Termination is the whole ballgame

If you take one thing from this chapter, take this: a language model has no concept of "done," so a loop without an explicit way to stop does not stop. It runs until something external kills it, and that something is usually your budget.

This is not a theoretical worry. Datadog's production data puts sixty percent of LLM failures in production down to rate-limit errors, and the cause is overwhelmingly runaway agent loops, an agent hits a failing tool call, retries with no hard stop, and exhausts your API capacity in minutes. The single most expensive bug in agentic systems is a loop nobody told how to quit.

So you build the exits in layers, because no single one is enough on its own.

A **verified goal** is the exit you want to hit: the checker confirms the task is done, and the loop ends because it succeeded. This is the good ending, and everything else on this list is a way to fail safely when you do not reach it.

An **iteration cap** is the blunt backstop: a hard maximum number of turns, after which the loop stops no matter what. Crude, essential, and the one that saves you at 3 a.m.

**No-progress detection** is the subtle one and the one that catches the nastiest failures. Watch whether the loop's state is actually changing. If the agent calls the same tool with the same arguments three times running, or the last few steps have not moved the world at all, the loop is stuck in place and should break, even though it has not hit its iteration cap and is still burning tokens with apparent enthusiasm. A loop that is busy is not the same as a loop that is progressing.

Real systems layer these: the loop exits happily on a verified goal, and failing that, it is caught by whichever of the cap or the no-progress detector trips first. Design these on day one. Every one of them is a lesson someone learned by shipping without it.

## TODO: running example

> **TODO (author):** the running-example section goes here, to be written once the running example is settled.
>
> *What this section should do:* Introduce the top-level loop the running example runs on every trigger, the reason-act-observe cycle it repeats while working a task. Show the five parts concretely: the verifiable goal (the alerting condition clears), the per-turn context, the tools and their honestly-fed-back results, the termination policy, and the error strategy. Make termination vivid, an incident agent stuck retrying a failing remediation is exactly the runaway loop that eats an API budget at 3 a.m., so show the iteration cap and the no-progress detector all wired in. Name the loop shape it uses (a ReAct loop with a human gate on remediation), and set up how the rest of Part III will deepen it.

## In the wild

The frameworks have converged on treating the loop as a first-class thing you design, and Google's ADK and LangGraph are the clearest pair. ADK gives you a `LoopAgent` whose whole job is to run its sub-agents around a cycle until a termination condition is met, and it is blunt about the lesson of this chapter: the loop primitive does not itself know when to stop, so you must supply the exit, either a `max_iterations` cap or a sub-agent that signals completion. The verified goal and the iteration cap, sitting right there in the API. LangGraph comes at it from the other side, modeling an agent as an explicit graph of nodes and edges so the loop's control flow, its branches, its cycles, its stopping conditions, is a thing you design and can see rather than an emergent property of a prompt. ADK 2.0 moved to the same graph-based model, and the convergence is the signal: making the loop's structure explicit is where the whole field has landed. ADK also frames its agents as a hierarchy "much like a company's organizational chart," the loop being one of a few composition primitives you assemble the org out of, which is the arc of this chapter in a framework.

The coding agents make the same case from production rather than from an API. Claude Code and the Codex-style agents are, structurally, a long ReAct loop wrapped in serious engineering: a verifiable goal (the tests pass), honest observation (compiler and test output fed straight back), and hard exits. They earn their reliability from how the loop is built around a model comparable to everyone else's. And the production guidance across the field has converged to the point of monotony: give the loop a testable goal, feed errors back as observations, and build layered termination from day one. The interesting arguments are no longer about whether to do these things. They are about how, and those arguments are the rest of Part III.

## Which phase

This is the **loop**, the fourth phase from Chapter 3, and this chapter is where it stops being one item in a list and becomes the discipline the rest of the part unfolds. Prompt, context, and harness were about building an agent that can take a good step. The loop is about taking step after step after step, adapting each time, and knowing when to stop. Everything from here forward, deterministic workflows, the non-deterministic split, multi-agent orchestration, validation, self-improvement, is about closing one kind of loop well, until the loops start handing work to each other and the org chart from the top of this chapter finally assembles itself.

---

### Author notes (not for the reader)
- **Running example (standing decision):** running-example section is a TODO placeholder pending the author's decision on the example. Same convention for all future chapters.
- **Chapter role:** this is the Part III opener and a *framing* chapter, so it is deliberately more prose than pattern-catalog. Renamed from "Loop Engineering" to "Composing Agents" (author's call): "loop engineering" is a 2026 coinage and datable, so the title now names the durable concept (composition) and the coinage is demoted to a term-of-the-moment in the prose. Opener reframed per author (option C): lead with the multi-agent org-chart *vision* as the hook, then pivot to "every box is a loop, so build one first," preserving the earn-your-way-to-multi-agent learning order while giving the part a multi-agent opening. The org chart bookends the chapter (opener + which-phase close). The named items are loop *shapes* (ReAct, Plan-Then-Execute, Reflection, Orchestrated, Human-Gated) and termination *controls* (verified goal, iteration cap, budget ledger, no-progress detection), given noun handles per the naming rule. Later Part III chapters go back to full catalogs.
- **Opener variety:** a *reframe/definition* opener ("the heartbeat"), not a failure or scene opener. Ch 7 reframe, Ch 8 scene, Ch 9 scene, so a reframe here breaks the run of two scenes. The one bold turn: "The single most expensive bug in agentic systems is a loop nobody told how to quit."
- **The spine:** (1) the loop is what turns parts into an agent; (2) a loop is not a chain (who chooses the next step is the fork the whole part lives on); (3) termination is the whole ballgame.
- **Sources (dated bucket):** ReAct (Yao et al. 2022), Reflexion (Shinn et al. 2023) for the lineage. The five-parts-of-a-loop framing (goal, context, actions, termination, errors) and the loop-vs-chain distinction from the 2026 loop-engineering guides (HappyCapy, Tosea, DataScienceDojo, Pexo). Datadog "60% of production LLM failures are rate-limit errors from runaway loops", verify the exact figure and attribution before print, it is the anchor stat. Layered termination (verified goal / iteration cap / budget / no-progress) is consistent across every 2026 source. "errors as observations" from CyberRaya's failure taxonomy (runaway, amnesiac, repeater, hallucinated-tool, silent-fail). LangGraph graph-as-control-flow.
- **Cross-refs:** four phases (Ch 3), context per turn (Ch 5), tools + honest error returns (Ch 7), human-gated loop (Ch 8), verifiable goal ties to eval (Ch 9), deterministic side (Ch 11), the split (Ch 12), orchestrated/multi-agent (Ch 13), validation (Ch 14), reflection-to-self-improvement (Ch 15). Renumber-check if the outline shifts.
- **In the wild balance:** now leads with Google ADK (`LoopAgent`, `max_iterations` / exit-signal termination, the org-chart hierarchy framing), then coding agents (Claude Code / Codex-style) and LangGraph, then the converged field guidance. Google-first is now a standing rule (voice guide); apply to every future chapter's *In the wild*.
- **Deliberately deferred:** deterministic-vs-nondeterministic gets its own two chapters (11-12), so this chapter only plants the fork and does not resolve it. Multi-agent kept to one paragraph with the standing "don't reach for it first" warning; full treatment in Ch 13.
- **Punctuation:** no em-dashes; a couple of ellipses. Loop-shape and termination names bold; the chapter uses fewer bold handles than a catalog chapter, on purpose.
