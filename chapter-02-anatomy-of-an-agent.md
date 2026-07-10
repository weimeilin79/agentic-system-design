# Chapter 2: The Anatomy of an Agent

*Draft in the author's voice. Register: a light voiced opener (Chapter 1 already ran hot, so this one comes in cooler), clean through the anatomy tour and model selection, voiced again in the Design Debate and the close.*

---

## The same diagram, every time

Sit through enough vendor talks and you start to notice something funny. Everybody's architecture slide is the same slide.

Seven or eight boxes. A model in the middle. Some tools hanging off one side, a memory store off the other, a box for guardrails, a box for orchestration, arrows connecting all of it. Swap the logo in the corner and AWS's diagram is Google's diagram is Microsoft's diagram is the one a startup drew on a napkin last week. For an industry that spends a lot of energy insisting its platform is different, we all drew the same animal.

I used to find that annoying. Now I find it reassuring. When five companies competing for the same money independently converge on the same shape, the shape is probably real, not marketing. It means there's an actual anatomy here, a set of parts an agent needs to be an agent, and it's stable enough to build a book on.

But that slide also plays a quiet trick on you, and untangling it is the whole point of this chapter. It draws the agent and the world around the agent as if they were the same thing. They aren't. So let's stop looking at the slide, and start with one agent, one body, before we ever get to a crowd of them.

## The anatomy

I'm going to use the body as a scaffold, because it maps cleanly and it'll keep the pieces straight in your head. Just don't take the metaphor to church; it's a teaching aid, not a truth. And here's the thing that surprises people: a single agent's body is a short list.

**The brain: the reasoning model.** The one part everyone thinks of first, and the part you'll obsess over least once you're past the demo. It looks at the situation and decides what to do next. It plans, it chooses what to reach for, it interprets what comes back. It is genuinely remarkable, and on its own it is also a brain in a jar... it can *think* about the world but it can't see it, touch it, or remember yesterday. Every other part exists to fix that.

**The senses: context.** The agent's eyes and ears on the world: the instruction, the relevant documents, the recent state, whatever you've placed in front of it this turn. This is the single biggest lever on quality, and it gets a whole chapter (Chapter 5), because the difference between a useful agent and a confidently useless one is usually not the brain. It's what the brain was allowed to see.

**The hands and feet: tools.** How the agent acts on the world, and how it moves through it: a query that returns the real metric, an API that actually files the ticket, a function that actually restarts the service, a step that carries it from one system to the next. This is also how it *grounds* itself in real readings instead of guessing. Without hands, the brain can only produce suggestions. The leap from "answers questions" to "does the job" lives here (Chapter 7).

**Memory: what it carries across time.** The senses are this turn; memory is every turn before. It comes in more flavors than people expect: the working scratchpad, the record of what happened, the store of facts, the learned procedures. Getting it wrong is its own rich source of pain, so it gets its own chapter too (Chapter 6).

**The voice: how it talks to other agents.** The part a solo agent barely needs and a crowd of them can't live without: the ability to send a message to a peer and hear one back. When agents hand work to each other, this is the mouth and the ear that make it possible, and it's the reason protocols for agent-to-agent conversation exist at all (Chapter 4, Chapter 14). Alone, an agent can leave this vestigial. The moment there are two of them, it's load-bearing.

Five parts. Only one of them is the model everyone fixates on. Hold onto that, because there's a sharper point coming, and it's the reason this chapter has a second half.

## When agents work together

So that's a single agent: a brain, some senses, hands and feet, a memory, and a voice. A capable little thing. But there's a ceiling on what any one body can do alone, and humans worked out where it is a long time ago. We didn't get anywhere by growing smarter individuals. We got somewhere by working together: dividing the labor, specializing, checking each other's work. Agents are on the same path, and the real power shows up when you put several of them side by side. The moment you do, a new set of concerns comes to the front. These are the boxes I left off the single body earlier: orchestration, identity and guardrails, observability. They're what a group of agents needs to work together without falling over.

**Coordination between bodies: orchestration.** A single agent has its own inner rhythm, the loop it runs to get one task done, and that rhythm matters enough to be the whole of the next chapter. But coordinating *many* agents is a different thing entirely. It's the choreography of a team, and it only exists once there's more than one of them (Chapter 14).

**Identity and guardrails.** What an agent is allowed to be and touch is meaningless in an empty room. It matters only because there are other actors, other systems, and things worth protecting. Identity is how a population of agents knows who is who; guardrails are the rules that population runs under. View it as a social contract, and it settles two questions. What it can touch: which tools it's allowed to reach for, and which slice of the toolbox its role even lets it see. Who it can talk to: which agents it can call, and which ones (the ones touching payroll, say) are off-limits. Identity decides both (Chapter 8, Chapter 9).

**Observability.** This one is a little different from the other two, because it starts inside the agent. Every agent should be built to report on itself: think of a fitness tracker on its own wrist, a running readout of what it decided and why. Put many agents together and those individual readouts add up to something like the CCTV overhead, every body in frame at once. And when something goes wrong, and something will, that footage is what a detective replays to work out exactly where it went sideways. So observability lives at both levels: wired into each agent, and shared across all of them (Chapter 17).

So the model was never the whole story, and neither, really, is the body. **A single agent is a short list. Things only get a bit more complex once agents start working together: coordination, trust, oversight.** That complexity lives in the space between agents, not inside one, and it's why Part II of this book builds the body and Part III builds the society.

## Picking a brain

Still, you have to choose the model, so let's talk about it, briefly, because it's the decision that ages fastest.

The instinct is to reach for whatever tops this month's benchmark leaderboard. Resist it. Benchmarks measure whether a model can *answer*; you're building something that has to *act, in a loop, with tools*, and those are different skills. What you're actually shopping for is a model that "feels agentic," which comes down to three things you can test directly:

- **Tool-calling behavior.** When it needs a tool, does it reliably produce a clean, correctly-typed call, or does it improvise, hallucinate parameters, or narrate what it *would* do instead of doing it? This single quality separates models that can run an agent from models that can only talk about one.
- **Instruction-following and steerability.** When you tell it to stay in bounds, does it? Can you nudge its behavior with the prompt, or does it drift back to its own defaults the moment the conversation gets long? An agent runs for many turns; a model that forgets your instructions by turn six is a liability.
- **Graceful failure.** When a tool errors or the situation is ambiguous, does it stop and ask, or does it barrel ahead confidently in the wrong direction? You want a model that knows the shape of its own uncertainty.

Test those on *your* task, with *your* tools, not on a spec sheet. And design so that swapping the model later is a config change, not a rewrite. The model you pick today will not be the model you run in a year, and the good architectures treat the brain as replaceable. The rest of the body is what you're really building.

## Build our own, or reach for a framework

Which brings us to the question every team hits in week two: do we use a framework, or roll our own harness?

The honest answer is that a framework will get you to a running agent faster than anything else, and for prototyping that speed is worth a lot. Reach for one. But know what you're signing up for. Frameworks make the easy things trivial and, sometimes, the necessary things awkward. The more the framework decides for you, the less you understand about what your agent is actually doing when it misbehaves at 2 a.m. And you *will* need to understand that.

There's a piece of advice from the people who build these things for a living that I've found holds up: frameworks are a fine place to start, but as you move toward production, don't be afraid to peel away abstraction layers and rebuild the parts you need to control with basic components. Start high-level to move fast; drop lower as the stakes rise. The framework is a scaffold, not a home.

Which is the abstraction question exactly, and it's contested enough to deserve its own sidebar.

## ⚖ Design Debate: How much framework?

There are two honest philosophies here, and the real question isn't which company you trust. It's how much of the agentic loop you want to own.

**Own the loop.** One camp keeps things thin: few abstractions, the agent's execution close to the model's natural behavior, the harness hand-written so nothing hides. The argument is that agents are simple at their core (a model, some tools, a loop), abstractions leak, and every layer between you and the model is a layer you can't debug. As models get smarter, they need *less* scaffolding, not more.

**Govern the loop.** The other camp hands you the loop and asks you to steer it: a managed runtime that already knows how to plan, call tools in a sandbox, spin up subagents, and enforce policy, so you configure and supervise rather than build. The argument is that most teams don't want to hand-roll durability, identity, and observability for every agent, and a runtime that ships with all of it is how this gets reliable and repeatable at scale.

Here's the part the vendor talks obscure: these are approaches, not tribes, and most of the big players sell you the whole ladder. Google is the clearest example. You can call the Gemini API raw and own everything. You can pick up the Agent Development Kit and build your own loop with your own routing. You can reach for the Antigravity SDK, which hands you a pre-built agent loop running on a Go harness and leaves you to govern it with tools, hooks, and safety policies. Or you can deploy onto the managed platform and let Google run the whole thing. OpenAI walks a similar path from the raw Responses API up through its Agents SDK, and Microsoft and Anthropic each ship their own thin and managed ends. The choice was never really Company A versus Company B. It's which rung you stand on.

So pick by how much of the loop you need in your hands. If you're a small team building one deep, custom agent and you have to understand every decision it makes, own the loop. The control is worth the plumbing. If you're standing up many agents and you need consistency, governance, and people who can hand work off to each other, let a runtime govern the loop and spend your attention on the parts that are actually yours.

My one firm rule is the anti-dogma one: don't grab the heaviest runtime to feel safe, and don't hand-roll everything to feel clever. Pick the least machinery that still lets you understand and control what your agent does, and be willing to move up or down the ladder as the stakes change. We come back to this in Chapter 11, when the loop gets real.

## What the anatomy can't show you

Here's the thing the diagram, for all its tidy boxes, can't tell you.

An anatomy is a body on a table. It names the parts, and naming them matters: you can't build what you can't see. But a body on a table isn't *alive*. Knowing that an agent has a brain and senses and hands tells you nothing about how it *moves*: how a single spark of reasoning becomes a step, becomes an action, becomes a checked result, becomes the next step, over and over, until the work is actually done.

That motion is the whole game. It's where reliable agents and impressive-demos-that-fall-over part ways. And it's four words long.

Turn the page.

---

### Author notes (not for the reader)
- **Running example (standing decision):** the running-example section is a TODO placeholder pending the author's decision on the example itself. Original Aegis prose is preserved in `aegis-sections-stash.md`. Apply the same convention to all future chapters: draft the chapter, mark the running-example section as TODO with a note on what it should do.
- **Restructure (your call):** the anatomy is now split into two levels. The single agent's **anatomy** is five parts (brain, senses, hands and feet, memory, voice), and orchestration / identity+guardrails / observability come to the front in "When agents work together". Note that observability is the partial exception: it's built into each agent (self-reporting) and also aggregates across the group, so its bullet flags that it lives at both levels rather than being purely group-level. Word "organ" retired in favor of "anatomy" / "parts" throughout.
- **The loop moved cleanly:** a single agent's own loop is its inner rhythm, deferred to Chapter 3 (the close still teases it). Coordination *between* agents is the separate, society-level thing that points to Chapter 14. So "the loop" and "orchestration" are no longer bundled.
- **The voice part** maps to agent-to-agent communication (A2A), flagged as vestigial for a solo agent and load-bearing the moment there are two. Points to Chapter 4 and Chapter 14.
- **Metaphor:** body-based, per your preference. Robot / teammate / kitchen versions are parked if you revisit.
- **Punctuation:** no em-dashes; a few ellipses where a trailing pause fits.
- **Thesis (softened per your note):** not "everything genuinely hard is between agents" (overstated) but "a single agent is a short list; things get a bit more complex once agents work together." Keeps the body-then-society, Part II-then-Part III payoff without overselling the difficulty.
- **Abstraction debate reframed (your Antigravity SDK catch):** dropped the too-tidy "OpenAI/Anthropic thin vs. Microsoft/Google managed" split, which is false, since Google spans the whole spectrum (Gemini API, to ADK, to Antigravity SDK, to managed platform). It's now an "own the loop vs. govern the loop" ladder that each vendor sells end to end, with Antigravity SDK as the concrete "handed a loop to govern" example. More accurate, and it reinforces the anti-dogma stance.
- **Ng's four patterns** still not named in prose; say the word and I'll add a short placement aside.
