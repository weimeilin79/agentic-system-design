# Chapter 8: Human-in-the-Loop Patterns

*Draft in the author's voice. Part II pattern chapter, same catalog format as Chapters 5 to 7. Running-example section left as a TODO block per standing convention. No em-dashes; a few ellipses. Which phase: the loop, enforced in the harness.*

---

## Nobody reads the diff

Here is a system I have watched get built more than once. The agent proposes an action. A human approves it. Everyone signs off on the design because there is a human in the loop, and a human in the loop means it is safe.

Six weeks later, the approval queue has four hundred items in it, the reviewer is three days behind, and she has developed a technique. She reads the title. She clicks approve. She has approved two thousand of these and the agent was right two thousand times, so she is not being lazy, she is being rational... right up until the one that would have deleted the production database, which she also approves, at 4:40 on a Friday.

The human was in the loop the whole time. The loop just was not doing anything. A gate that is always approved is not a control, it is a liability with a signature on it, and the signature makes it worse, because now a person is accountable for a decision they were never actually in a position to make.

So set aside the comforting idea that human oversight is something you add. Oversight is something you *design*, with the same rigor you would give a retry policy, and most of this chapter is about protecting the scarcest resource in your system, which is not tokens or GPUs but the finite attention of the person you keep interrupting.

## Reversibility is the axis

The instinct is to gate by importance, and importance is the wrong axis. Plenty of important actions are perfectly safe to automate, because if the agent gets them wrong you simply do them again.

The axis that matters is **reversibility**, crossed with **blast radius**. Reading a dashboard is free to undo, nobody needs to approve it. Drafting a message is trivially reversible, you delete the draft. Posting that message to ten thousand customers is not reversible in any way that matters, and neither is deleting a record, moving money, or deploying to production. The question is never "is this action important." It is "if this is wrong, what does it cost to be wrong, and can we take it back?"

This gives you tiers rather than a switch, and the tiers are the whole design. Low-risk, reversible actions run autonomously and get logged. Medium-risk actions run autonomously but surface for after-the-fact review, so a human sees what happened without standing in the way. High-risk actions stop and wait. Irreversible, high-blast-radius actions always stop and wait, no exceptions, no confidence threshold high enough to skip the gate.

Notice what that buys you. The reviewer's queue now contains only things that genuinely need her, which is the only way her approval means anything at all.

## How human oversight breaks

Five failure modes, and you will recognize most of them.

- **Rubber-stamping:** the queue outgrows the reviewer, and approval becomes theatre. This is not a discipline problem, it is a design problem, and the fix is never "review faster."
- **Gating the trivia:** approval required to read a calendar or classify an email. This burns attention on nothing and trains everyone to ignore the queue, which is how the important item gets missed.
- **Blocking the loop:** the agent holds a synchronous connection open while it waits for a human, and the infrastructure kills it.
- **Stale approval:** the human approves an action based on a world that has since moved on.
- **The silent override:** a reviewer edits the agent's proposal into something correct, and that correction goes precisely nowhere.

## The HITL patterns

Thirteen patterns, in the order you meet them: decide where the gate goes, make the pause survivable, make the human's decision fast and right, and learn from every call they make.

**Tier by reversibility.** *Sort every action the agent can take by what it costs to undo, and gate accordingly.* Before you write a line of interrupt code, enumerate the agent's actions and put each in a tier: autonomous with logging, autonomous with after-the-fact review, gated, or always gated. This is a boring afternoon with a spreadsheet and it is the highest-leverage work in the chapter, because every later decision follows from it. Do it in plain language, with the people who will be accountable when it goes wrong.

**Put the gate in the orchestrator, not the model.** *The workflow decides when a human is needed, not the agent's judgment in the moment.* It is tempting to hand the model a `request_human_review` tool and let it decide when to call it. Resist that for anything high-stakes. A non-deterministic reasoner will eventually feel confident and skip the gate, and it will feel most confident in exactly the novel situation that most needed a human. Gates on irreversible actions belong in deterministic orchestration code (Chapter 11), where they fire every time regardless of how the model feels. Let the model propose. Let the workflow decide who approves.

**Escalate on thresholds.** *Route the marginal cases to a person using explicit rules on value, anomaly, and confidence.* Between "always autonomous" and "always gated" sits the large middle where the right answer depends on the instance, and the way you navigate it is with thresholds: the dollar value of the transaction, whether the request looks like anything the agent has seen before, and the confidence score from the next pattern. Thresholds are what let you avoid the false choice between full autonomy and full review, and they are the dial you will spend the most time tuning. Keep them in plain, declarative rules you can read and audit, not buried in the model's judgment.


**Rate the confidence, and let it route.** *Have the agent score how sure it is, and use that number to decide what runs and what waits.* A gate that fires on every action of a given type is blunt; a gate that fires on the *uncertain* ones is sharp. Ask the agent to attach a calibrated confidence to each proposed action, then wire that score into the tiers: high confidence on a low-risk action runs, low confidence on anything routes to a human, and the same action can go either way depending on how sure the agent is. The catch is that raw model confidence is often poorly calibrated, so treat the number as one signal among several (value, anomaly, novelty), and check it against outcomes over time rather than trusting it on faith.

**Checkpoint before you wait.** *Serialize the agent's state to durable storage, then release the request.* This one is physics, not preference. Humans do not answer in seconds, and your whole stack assumes something will: API gateways close connections after about thirty seconds, serverless functions time out in ten seconds to a few minutes, and OAuth tokens expire mid-wait, some in half an hour, some in an hour or two, so a held-open action fails on resume even after approval. So do not hold anything open. The agent writes its state to a durable checkpoint, the pending action enters a queue with a time to live, and the request ends. Every serious framework now ships this rather than making you build it, LangGraph, for one, offers both compile-time breakpoints and runtime interrupts that persist graph state and resume on command. The durable-execution machinery underneath is a harness concern (Chapter 18).

**Resume from the checkpoint, not the top.** *When the decision arrives, continue exactly where the agent paused.* The payoff of checkpointing is that approval restarts nothing. The decision lands, the orchestrator loads the saved state, and execution picks up from the paused step with the human's verdict folded in. No re-reasoning, no repeated tool calls, no duplicated side effects from replaying half the run. If your "resume" actually re-runs the agent from the beginning, you have a retry, not a resume, and retries on an agent that already took actions are how you double-charge a customer.

**Re-validate against the world before acting.** *Approval was granted against a snapshot; confirm that snapshot still holds.* An approval that sat in a queue for two hours is a statement about a world that may have moved on. The incident may be resolved, the record may have changed, the deploy may have been superseded. Before executing an approved action, re-read the state it depended on and confirm it still makes sense. If it does not, do not execute it, and do not silently adapt it either. Go back to the human.

**Set a timeout, and decide which way it fails.** *Every pending approval needs an expiry and a default.* What happens when nobody answers? If the answer is "the agent waits forever," you have built a system that quietly stalls. Give every gate an SLA and an explicit expiry behavior: auto-reject (safe, and the default for irreversible actions), auto-escalate to a second approver, or, in genuinely low-risk cases, auto-proceed. Decide this deliberately per tier, not during an incident.


**Write an evidence pack, not an alert.** *Give the reviewer everything needed to decide in a few seconds, and nothing else.* A notification that says "Agent wants to run remediation, approve?" is an invitation to rubber-stamp, because deciding well would take ten minutes of digging that nobody has. Instead, hand the reviewer a compressed brief: the proposed action, exactly what it will change, the reasoning and evidence that led here, the agent's confidence, which policy the action touches, and what happens if it is wrong. Aim for a decision in under fifteen seconds without the reviewer leaving the interface. This is the single highest-return investment in a human-in-the-loop system, and it is the one teams skip, because building a good review surface feels like it is not the real work. It is the real work. The quality of your oversight is capped by the quality of what you show the reviewer.

**Offer four verbs, not two.** *Approve, edit, reject, and respond.* A binary approve-or-reject throws away the most useful thing a human does, which is fixing the proposal. Let the reviewer edit the action before it runs (the agent had the shape right and the parameters wrong), or reject it, or reply with a question or a correction that sends the agent back to reasoning with new information. Each verb carries different information, and a system that only records "approved" or "denied" has thrown the other two away.

**Meet people where they already work.** *Put the gate in Slack, in the pull request, in the ticket, not in a bespoke console nobody opens.* An approval queue on a dashboard is an approval queue that gets checked on Tuesdays. Route the decision to the surface where the reviewer already lives, and the trigger too. Coding agents that land their work as pull requests get this for free: the review surface is the one engineers already use, with diffs, comments, and history built in, and the approval gate is just a merge.


**Capture every override.** *An edit or a rejection is a labeled training example, an audit record, and a bug report all at once.* When a reviewer changes what the agent proposed, that correction is the most valuable signal your system produces, and in most deployments it evaporates the moment the button is clicked. Store every approval, edit, rejection, and reason, attributable and replayable. It is your audit trail when someone asks who authorized what. It is your evaluation set (Chapter 9), because those are precisely the cases the agent got wrong. And a pattern in the overrides tells you where the agent is systematically weak, which is a bug report you did not have to write.

**Earn autonomy with evidence.** *Start with tight gates, and loosen them on data rather than optimism.* The rule of thumb worth stealing: launch more conservative than feels necessary, watch the queue, and relax a gate only when a specific action has accumulated a real run of clean, human-approved executions behind it. Progressive autonomy is the right shape, because trust is per-action, not per-agent. The agent may have earned the right to restart a stateless service without asking, and not the right to touch the database, forever.

## From in-the-loop to on-the-loop

There is a ladder here, and knowing which rung you are on prevents most arguments.

*Human in the loop* means the agent stops and waits for a decision on this action. *Human on the loop* means the agent acts and the human supervises, watching a stream of actions and intervening when something looks wrong. *Human out of the loop* means the agent acts and nobody is watching in real time, which is fine for the reversible tier and reckless everywhere else.

The trajectory of a maturing system is up this ladder, one action type at a time, as evidence accumulates. But watch the arithmetic, because it is the bottleneck the whole industry is walking into. If an agent can propose faster than a human can review, then human-in-the-loop does not make the system safe, it makes the human the throughput limit and the safety theatre inevitable. The way out is not a faster reviewer. It is moving the reversible work off the queue entirely, so the human's attention lands only where it changes an outcome. This is the review-throughput problem, and it gets a chapter of its own (Chapter 27) because it is the shape of the next few years.

## TODO: running example

> **TODO (author):** the running-example section goes here, to be written once the running example is settled.
>
> *What this section should do:* Show the tiering in action on the running example. Diagnosis, log reading, and metric queries run autonomously (reversible, zero blast radius). Restarting a stateless service is medium risk, autonomous with after-the-fact review. Anything that writes to production, touches customer data, or changes a config is gated, always, no confidence threshold high enough to skip it. Show the async interrupt (checkpoint at 3 a.m., approval arrives at 3:04, re-validate that the incident is still live before executing), the evidence pack (proposed action, blast radius, reasoning, confidence, what breaks if wrong) landing in the on-call Slack channel, the four verbs (the on-call engineer edits the remediation rather than approving it as proposed), and the override being captured as an eval case. Close on the ladder: after two hundred clean service restarts, that action graduates to on-the-loop, while database writes never do.

## In the wild

The frameworks have converged on interrupt-and-resume, and Google's ADK is the fullest example. In ADK 2.0 (shipped for both Python and Go), human-in-the-loop is a first-class node in the graph rather than something you bolt on, built on tool confirmation: a tool is flagged as sensitive (a `send_wire_transfer` pauses, a plain search does not), and when the agent calls it the run suspends and surfaces its state to a human, showing what led here, why the tool was requested, and the predicted outcome, which is the evidence pack by another name. The pause is durable: the run state lives in the session, so a workflow can resume minutes or days later, after a process restart, even across a different runtime, with the confirmed call injected back into context so the model does not loop on it. Resume is idempotent, every node can carry a timeout and a retry policy, and Gemini can be prompted to emit an explicit confidence score to drive escalation, which is the confidence-rating pattern shipped in the platform. LangGraph does the same shape from the other direction, with compile-time breakpoints and a runtime `interrupt()` that persists graph state and resumes on a command rather than replaying from the start.

Coding agents made the cleanest version of the argument almost by accident. A background coding agent that lands its work as pull requests gets human-in-the-loop for free: the approval gate is a code review, the evidence pack is a diff, and the four verbs are the ones engineers have used for twenty years. Nobody has to build a review console, and nobody rubber-stamps a diff the way they rubber-stamp a notification, because the surface itself invites reading. The large streaming and consumer-software companies that run fleets of these agents lean on exactly this, the review surface predates the agent, so oversight came ready-made.

Beyond code, the pattern is now standard in high-stakes domains for the obvious reason: the actions are irreversible. Financial and treasury agents read balances and propose transactions but cannot sign or broadcast, routing every state-changing call to a human or a multi-party approval flow. Customer-operations agents handle routine queries autonomously and escalate on confidence, complexity, or an explicit request, handing the human the full session as a context package so nothing is re-explained. And the enterprise maturity models all point the same direction, from human-in-the-loop toward human-on-the-loop, deliberately, one action at a time, rather than in a leap.

## When not to reach for these

Human oversight has a real cost: latency, reviewer attention, and operational machinery. On low-stakes, reversible actions, a gate buys nothing and costs everything, so let those run and log them. If you find yourself building an approval workflow for an agent that only reads and summarizes, stop. And if a gate exists purely so that a human's name is attached when things go wrong, that is not oversight, it is liability laundering, and it is worse than no gate because it manufactures the illusion of control. Either give the reviewer what they need to genuinely decide, or take the gate out and own the automation.

## Which phase

Human-in-the-loop is **loop** work: it is a decision about how the agent's cycle pauses, yields, and resumes, which is why it sits at the phase where reliability lives. But it leans hard on the **harness** for the machinery, the checkpointing and durable execution that make an asynchronous pause survivable (Chapter 18), and the gates themselves are ultimately enforced as policy, not as instructions in a prompt (Chapter 20). A guardrail the model can talk its way past was never a guardrail.

---

### Author notes (not for the reader)
- **Running example (standing decision):** the running-example section is a TODO placeholder pending the author's decision on the example itself. Apply the same convention to all future chapters: draft the chapter, mark the running-example section as TODO with a note on what it should do.
- **Opener variety:** this is a *scene* opener (the reviewer who approves by title), not a failure opener, per the voice guide's rotation rule. Chapter 7 was a reframe, Chapter 6 a scene, Chapter 5 a reframe. The one bold turn: "a gate that is always approved is not a control, it is a liability with a signature on it."
- **The spine:** reversibility, not importance, is the gating axis; and the scarce resource is reviewer attention, not compute. Everything else follows. The "liability laundering" line in *when not to* is the deliberate counterpart to the opener.
- **Patterns and sources (dated bucket):** tiering by reversibility and the four-tier ladder (T1-2 autonomous with logging, T3 review, T4 always gated) from 2026 HITL escalation-design writeups. Orchestrator-driven rather than LLM-driven gates (the model "feels confident" and skips) from the Spring AI checkpoint pattern write-up. Async interruption constraints are concrete and worth verifying: API gateway ~29s connection close, serverless ~10 to 300s timeouts, OAuth expiry (HubSpot ~30m, Google ~1h, Salesforce ~2h), stale cursors/snapshots requiring re-validation. LangGraph static breakpoints + dynamic `interrupt()` with checkpoint store. Evidence packs (sub-15-second decisions) and the approve/edit/reject/respond verbs from enterprise HITL blueprints. Rubber-stamping / "theatre of approval," gating trivia, and the missing feedback loop are the three named killers. Progressive autonomy ("loosen only after a run of clean executions," one source says ~200) and the in-the-loop to on-the-loop maturity ladder. Verify all specifics before print.
- **Cross-refs:** deterministic orchestration (Ch 11), multi-agent (Ch 13), durable execution and checkpointing (Ch 18), observability (Ch 19), policy and guardrails (Ch 20), evaluation set from overrides (Ch 9), review-throughput bottleneck (Ch 27), Honk teardown (Ch 25). Renumber-check if the outline shifts again.
- **Deliberately not covered:** the UI design of review surfaces beyond the evidence-pack principle, and the org/accountability question (who is on the hook), which belongs in Ch 27.
- **In the wild (reordered on request):** Google ADK 2.0 leads (Python and Go), HITL as a first-class graph node built on tool confirmation, durable session-based resume across restarts/runtimes, idempotent resume, per-node timeout/retry, Gemini confidence-based escalation. LangGraph `interrupt()` + breakpoints as counterpart. The PR-as-review-surface point is now generic (background coding agents at large streaming/consumer-software companies) rather than naming Honk/Spotify, per author. Added two more domains: financial/treasury agents (read-only auto, state-changing gated / multi-party approval, "cannot sign or broadcast") and customer-operations agents (escalate on confidence/complexity, full session as context package). Sources: ADK Go 2.0 and Java 1.0 Google Developers Blog posts, ADK action-confirmation docs, 2026 enterprise-ADK and HITL writeups. Google now leads; no single-vendor tilt since Ch 7 also carried Google, watch overall balance across the book. Verify specifics before print.
- **Punctuation:** no em-dashes; a couple of ellipses. Pattern names bold, intents italic. No *Targets* annotations, consistent with Ch 6 and 7.
