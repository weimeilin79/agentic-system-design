# Chapter 4: Multi-Model Patterns

*Draft in the author's voice. First chapter of Part II. Register: a medium-warm reframe opener (Chapter 3 ran hot, so this comes down a notch), clean through the mechanics, warmer again at the close. No em-dashes. Which phase: harness, with a foot in the loop.*

---

## There is no best model

Every few weeks a new model tops the leaderboard, and every few weeks someone on your team will want to rip out the old one and wire in the new champion. I get the urge. I've had it. But it rests on a quiet wrong assumption: that there is a best model, singular, and your job is to find it and run it everywhere.

There isn't. There's only the right model for the job in front of it, and a real agent does a dozen different jobs in a single run. It classifies an alert. It decides which runbook is relevant. It reasons through a messy root cause. It summarizes what it did. Those are not the same task, and they do not want the same model. Handing all of them to whatever sits at the top of this month's leaderboard is like hiring one brilliant, expensive senior engineer and making them also answer the phones.

So the question this chapter answers isn't "which model should I use." It's "which model, for this step, right now." That reframe is the whole pattern.

## One agent, many models

Here's the shift. In Chapter 2 I told you to treat the model as a swappable part, and in Chapter 3 I put model routing squarely in the harness. Now we cash that in. An agent is not one model with a wrapper around it. It's a harness that reaches for whichever model fits the step it's on, the way you reach for a different tool for a different screw.

The instinct to standardize on one model comes from a good place. One model is simpler. One bill, one API, one set of quirks to learn. Start there, honestly, every time. But as the agent grows and the bill grows with it, you'll find that a surprising amount of what it does is cheap, repetitive work that a small model handles perfectly, and a small slice is genuinely hard reasoning that only a frontier model gets right. Paying frontier prices for the cheap work is the single most common way agent costs quietly balloon.

## Matching model to task

The core skill here is reading a step and asking what actually constrains it. Most steps are bound by one thing, and that one thing points straight at the model.

**When cost and volume dominate.** The step is simple and runs constantly: classify this alert into one of five categories, extract these fields, decide whether this text is a question or a command. Reach for a small, cheap model. The task is narrow enough that a small model is not just good enough, it's often *better*, because there's less room to overthink, and at high volume the cost of a big model is money set on fire.

**When latency dominates, especially at the edge.** The step has to answer *now*: on-device, on-box, in the middle of a real-time interaction, anywhere a round trip to a big hosted model is too slow or the hardware simply can't hold a large one. Here speed is the whole game. A small model that generalizes well enough and responds instantly beats a smarter model whose better answer arrives too late to matter. Small and close wins, and sometimes it's the only thing that fits.

**When the reasoning is genuinely hard.** The step is ambiguous or multi-step: three plausible causes to weigh, a remediation to plan, a real trade-off to make. A too-small model gives you confident nonsense here, so these want more horsepower, but reaching straight for the biggest, priciest frontier model is usually overkill. What they mostly want is a strong general model with its *thinking* turned up: dial reasoning effort or hand it a bigger thinking budget. More thinking on a solid mid-tier model usually beats paying frontier sticker price on every hard call. Save the true frontier tier, the most capable and most expensive, for genuinely frontier problems: research-grade math, novel science, the open-ended reasoning a research or discovery app is built around. Most production agents rarely touch work that hard.

**When the task is specialized.** Generating a patch, reading a dashboard screenshot, retrieving from a store: a code model, a vision model, an embeddings model. A specialist that's a fraction of the size will often beat a giant generalist on its home turf.

The mistake is uniformity in any direction. Frontier-everywhere burns money on trivia and adds latency where you can't afford it. Cheap-everywhere fails the moment the work gets hard. The craft is reading each step for what binds it, and matching the model to that.

## The token bill

You can't match models to steps without a feel for what they cost, and the pricing is stranger than it looks.

Start with the one rule that never changes: **output costs far more than input**, usually five to ten times more, because generating tokens is heavier than reading them. Agents are output-heavy by nature. They plan, they explain, they call tools in a loop. So your bill is driven by what the models *say*, not by what you send them. Cap your output lengths like you mean it.

As a snapshot at the time of writing, the frontier tiers land roughly here, per million tokens, input then output: Anthropic's Claude Opus around $5 / $25, Google's Gemini Pro around $2 / $12, OpenAI's GPT-5-class around $2.50 / $15. Every one of them also sells a budget tier ten to twenty times cheaper (Claude Haiku, Gemini Flash, the GPT mini and nano lines), which is exactly what makes the cheap-step, expensive-step split pay off. Those exact numbers will be stale by the time you read this. Prices have fallen 80 to 90 percent in two years and keep dropping. The shape holds even as the figures move.

Two things trip up almost everyone. The first is **reasoning tokens**: reasoning models think in hidden tokens you never see, and you pay for every one at the output rate, so a single call can burn tens of thousands of thinking tokens before it writes one paragraph. The real cost of a reasoning model can be several times its headline price. The second is that **cheaper per token is not cheaper per task**. Two models at the same rate can produce very different bills because one is chattier or tokenizes your text less efficiently. Measure tokens per task on your own workload before you trust anyone's rate card.

Then there's the lever that matters most for agents: **prompt caching**. An agent resends the same system prompt and much of the same context on every step of a loop, and all three providers will cache that repeated context and charge you roughly a tenth of the input price for the cached part. For a looping agent, turning caching on is often a bigger win than switching models. And for anything that doesn't need to be real-time, classification, summarization, evaluation runs, the batch APIs knock another 50 percent off in exchange for a slower turnaround.

## Routing and cascades

Once you've accepted many models, something has to decide which one runs. Two patterns do most of the work.

**Routing** puts a small, fast classifier in front: it looks at the incoming task and sends it to the right model. The router itself should be cheap and boring, because it runs on every request and any latency it adds is paid every single time. A fine-tuned small model or even a simple classifier is usually the right router. Do not put a frontier model in charge of deciding which model to use; that defeats the entire point.

**Cascades** go cheap-first and escalate. Try the small model. If it's confident and the answer passes a quick check, ship it. If it's uncertain, or the check fails, escalate to the bigger model. Most requests never reach the expensive tier, so you get frontier quality on the hard cases and small-model economics on the rest. The trick is the escalation signal: a confidence score, a validator, a cheap sanity check. That signal is a little loop, which is why this chapter has a foot in the loop phase even though it lives mostly in the harness.

Both patterns add a failure point and a little latency. Keep the routing logic simple enough that you can reason about it at 3 a.m., because eventually you'll have to.

## Build your own: fine-tuning open models

Sometimes the right model for a step isn't one you can buy. It's one you build.

When a task is narrow, high-volume, and stable, a fine-tuned open-weight model can beat a frontier API outright: cheaper per call by an order of magnitude, faster because it's smaller and closer, and private because the data never leaves your walls. That last one is decisive for a lot of teams. If you can't send incident logs or customer records to a third-party API, a model you host yourself stops being an optimization and becomes the only option.

The open field is deep now and moves monthly, so treat any specific name as a snapshot: as of this writing, families like Qwen, Llama, Mistral, Gemma, DeepSeek, GLM, and the GPT-OSS releases cover most needs, with licenses ranging from the very permissive (Apache 2.0, MIT) to the merely mostly-permissive (watch Llama's usage cap). By the time you read this the leaderboard will have turned over at least twice. The pattern outlives the names.

The technique that makes this practical is **parameter-efficient fine-tuning** (LoRA and QLoRA). You're not retraining a model from scratch. You're tweaking an existing open one so it fits your use case better and runs faster: teaching it your formats, your domain, your sense of a good answer, on a modest GPU budget. That's the whole move. Take a capable open base, adapt it to the narrow thing you do a thousand times a day, and you get a smaller, faster, cheaper model that's better at *your* job than any generalist.

Now the honest part, because this is where teams get hurt. Fine-tuning moves you from renting a model to running one, and running one means you now own serving (vLLM, SGLang, or Ollama for the small stuff), evaluation, monitoring, and drift. You own the GPUs, or the GPU bill.

And here's the trap people forget: the moment you fine-tune, you're pinned to that base model. Open models improve monthly, so the stronger base that lands next quarter does nothing for you until you redo your tuning on top of it. Your fine-tuned model starts aging the day you ship it. A frontier API upgrades under you for free; a model you tuned only upgrades when you put in the work again.

The rough economics: self-hosting a large model only pays off above real volume, and fine-tuning is only worth it once you have enough good examples and a stable enough task to justify the pipeline. Below that line, a frontier API is not the lazy choice. It's the correct one. Do not fine-tune to feel like a serious ML team. Fine-tune when the numbers say to.

## When a model isn't there

Models go down. They rate-limit you at the worst moment. They refuse a request that yesterday they answered. And the API ones get deprecated out from under you on the vendor's schedule, not yours.

For a system that matters, especially one that pages you at 3 a.m., a single model is a single point of failure. The fix is a fallback: a second model, ideally from a second provider, that the harness fails over to when the primary is unavailable. It doesn't have to be as good. It has to be *there*. An incident agent that goes dark because one provider is rate-limiting is worse than one running on its backup model.

This is also the quiet argument for the model-agnostic harness from Chapters 2 and 3. If your tools, memory, and logic live in the harness and the model is a config value, then failing over, swapping in next month's better model, or dropping in a fine-tuned one is a config change, not a rewrite. Build it that way and multi-model stops being scary and starts being a dial you turn.

## Aegis gets a bench of models

Let's wire this into the incident agent, because Aegis is exactly the kind of system that earns a bench of models rather than one.

- A **small fine-tuned open model**, hosted on-box, classifies each incoming alert into a type and severity. This runs on every single alert, the data is sensitive, and the task is narrow and stable. Perfect fine-tuning territory.
- An **embeddings model** pulls the relevant runbook and the most similar past incidents. Cheap, specialized, constant.
- A **strong general model, with its reasoning turned up**, does the actual root-cause reasoning, but only on the incidents that survive the cheap triage and look genuinely hard. Rare, more expensive, worth it.
- A **cheap general model** writes the human-readable incident summary at the end. Nobody needs a frontier model to write a tidy paragraph.
- And a **fallback model from a second provider** stands by, so a rate limit on the primary at 3 a.m. degrades Aegis instead of killing it.

One agent, five models, each matched to its step, all swappable behind the harness. That's the pattern in full.

## When not to reach for this

Start with one model. I mean it. A single capable model with a clean harness will take you a long way, and every model you add is another dependency, another failure point, another few milliseconds of routing, another thing to monitor. Multi-model is an optimization you earn, not a starting posture.

Don't add routing until cost or latency actually forces it. Don't fine-tune until the task is narrow, stable, and high-volume enough to repay the MLOps cost you're signing up for. For broad, changing, or low-volume work, a frontier API is the right default and will stay that way. The whole point of this chapter is to match the model to the job, and for a lot of jobs, "one good model" is the honest match.

## Which phase

This is mostly a **harness** pattern. Choosing models, routing between them, serving the ones you host, failing over when one dies: that's all runtime machinery, and it's why a model-agnostic harness pays off here more than anywhere. But it keeps a foot in the **loop**, because cascades and escalation run on feedback signals, a confidence score, a validator, a cheap check that decides whether to go again with a bigger model. Model selection and the loop are quietly the same move: try, check, escalate if you must.

---

### Author notes (not for the reader)
- **Reframe (from Multi-Modal):** per your call, this chapter is now about *when to use which model*, not mixed input modalities. Opener is "there is no best model," which reframes the leaderboard-chasing instinct.
- **Token bill section (your ask):** a dedicated pass on token usage across Claude, Gemini, and OpenAI: output costs 5-10x input, reasoning/thinking tokens as the hidden multiplier, "cheaper per token is not cheaper per task," and prompt caching (roughly a tenth of input price) as the single biggest lever for a looping agent, plus batch APIs at 50% off. Durable principles up front, dated numbers flagged.
- **Current as of drafting:** open-model families (Qwen, Llama, Mistral, Gemma, DeepSeek, GLM, GPT-OSS), LoRA/QLoRA, and the frontier token prices (Claude Opus ~$5/$25, Gemini Pro ~$2/$12, GPT-5-class ~$2.50/$15 per million in/out) are current per searches at draft time. All of it belongs in the quarantined/dated bucket, verify names and numbers before print.
- **Fine-tuning (your ask, then trimmed):** framed as adapting an open base with LoRA/QLoRA to fit your use case better and faster. Distillation was cut per your note (it's a different move). Added the warning you asked for: a fine-tuned model pins you to an aging base, while a frontier API upgrades under you for free. Privacy is the decisive driver; economics gate when *not* to.
- **Continuity:** cashes in Chapter 2's "the model is a swappable part" and Chapter 3's "model routing lives in the harness / model-agnostic harness." Aegis gets a five-model bench, extending the running example.
- **Which phase:** harness with a foot in the loop (escalation is a feedback decision), stated explicitly per the new recurring beat.
- **Register:** medium-warm reframe opener (Chapter 3 was hot), clean through matching/routing/fine-tuning mechanics, warmer at "when not to reach for this." No em-dashes; a couple of ellipses.
- **Open question:** Multi-Model sits as the first Part II chapter, but routing is a harness concern, so it could also live nearer Tooling (Ch 7). Left at position 4 per your instruction.
