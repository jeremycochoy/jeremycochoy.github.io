---
layout: post
title: "Effective AI Usage Patterns"
description: "Principles derived from real-world prompt data — 8 patterns for building a productive working relationship with AI."
author: Jeremy Cochoy
---

Most guides on "prompting" focus on wording tricks — how to phrase a question, what role to assign the AI. This guide is different. It comes from analyzing 500+ real prompts from engineers working with AI daily on production systems over several weeks. The patterns that emerged aren't about writing better sentences. They're about building a working relationship with a tool that is powerful, tireless, and confidently wrong in ways you won't expect.

These 8 principles are ordered as a learning progression:

- **Points 1–3** — Foundations: how to interact with the AI effectively
- **Points 4–5** — Scaling up: extending the AI beyond a simple question-answer tool
- **Points 6–8** — Staying in control: managing risk as you delegate more

The arc is: **use it → extend it → shape it → manage it → control it → think above it.**

---

## 1. Progressive delegation — verify before you commit scope

Don't open with "analyze all the data and write me a report." Start small: "Do you see the data?" Then: "Who are the users?" Then: "Analyze them." Each step verifies a capability before committing more scope.

**Example:**
A conversation following this pattern:
1. "Can you access the production database on the staging server?" — *can you even reach it?*
2. "Yes, use ssh deploy@staging" — *establishing capability*
3. "I want you to compare the last 3 deployments and flag any performance regressions" — *now the real task*
4. "Also include memory usage, not just latency" — *steering after seeing first output*
5. "Write this up as a summary report" — *committing to deliverable only after the content is validated*

**Why it works:**
Each step is a low-cost checkpoint. If step 1 fails (can't access the server), you haven't wasted time on a detailed analysis prompt. If step 3 produces a bad framework, you course-correct at step 4 before investing in formatting. The total cost is lower because failures are caught early.

**The anti-pattern:**
Writing a 500-token prompt that says "SSH to staging, query the database, compare the last 3 deployments across latency and memory, and write a formatted summary report." If any assumption is wrong (can't SSH, schema is different, there were 5 deployments not 3), the entire effort is wasted.

---

## 2. Context is an asset — invest in it

Keeping a conversation alive across hours or days is not laziness. It's an investment. Every prompt you've sent, every correction you've made, every piece of domain knowledge you've shared is now part of the AI's working memory. That accumulated context is what lets you steer with 3-token prompts instead of 400-token briefs.

**Example:**
A single session about migrating a legacy API spanned 12 days, 50+ prompts, across 8 distinct working days. By day 10, prompts were things like "Check the logs" (3 tokens), "Yes, deploy it" (4 tokens), "Rollback the auth service" (5 tokens). These are precise directives — not vague — because both sides share 12 days of accumulated context about which endpoints were migrated, what broke, what the rollback procedure is, and which services depend on what.

Compare with starting a fresh session each morning: you'd write 300-token prompts to re-explain the migration state, which services are done, which are pending, and what "rollback" means in this context.

**The trade-off:**
Cold start (new session) costs prompt tokens but avoids context pollution. Warm continuation (same session) is efficient but requires the context to still be relevant. The most effective pattern is to keep sessions alive for as long as the *problem* persists, and start fresh when the *problem* changes.

**How to apply:**
Don't close a session just because you're stepping away. Come back to it tomorrow. The AI remembers everything. The 30 seconds of re-reading your last exchange saves 5 minutes of re-prompting.

---

## 3. Correct the framing, not the output

When the AI produces an incorrect or unintended output, the instinct is to point at it and say "fix this."

**Example:**
The AI produced a prompt analysis that judged short prompts as a weakness ("would be useless as standalone documentation"). There were two options:

- **Correct the output:** "Short prompts aren't a weakness, they're efficient." — the AI adjusts that one judgment, but keeps evaluating prompts as if they were supposed to be self-contained instructions.
- **Correct the framing:** "Prompts aren't supposed to be documentation, they gave us a window on how the user use them and we want to understand this."

The first feels like it fixes the problem. But the AI still thinks prompts should be judged by conventional quality standards — it just makes an exception for length. The second — a single sentence — changed the entire analytical framework. The AI re-derived all conclusions from the corrected premise, including ones not yet produced.

**Why it works:**
The output is a symptom. The AI's mental model is the cause. Fixing the output patches one symptom; fixing the mental model fixes all downstream outputs at once.

**How to apply:**
When the AI gets something wrong, ask yourself: is the output wrong, or is its understanding of what I want wrong? If the latter, state what it should be thinking about, not what it should be writing.

---

## 4. Async operator pattern — the AI works while you don't

The AI can run background tasks, monitor processes, and check periodically. Treating it as a persistent operator — not a synchronous call-response tool — multiplies your effective working hours.

**Example:**
Patterns like:
- "Run the full regression suite. Check every 20 minutes — if a test fails, investigate but don't push any fix until I review."
- "Good morning. Did the overnight migration finish? Show me the error count."
- "Start the deployment. Monitor the health checks for 3 hours, and rollback if error rate exceeds 1%."

The user kicks off long-running tasks, then steps away. The AI monitors, and the user comes back to results.

**The rhythm:**
Burst of rapid interaction (5-10 prompts in 30 minutes to set up and steer) → async gap (hours or overnight, AI working) → brief check-in ("How is it going?", "Check on them") → either another burst or a final "merge and deploy."

**How to apply:**
When you have a long-running task (training, deployment, data processing), don't sit and wait. Tell the AI what to monitor and what to do if things go wrong, then come back later. The AI's time is cheaper than yours.

---

## 5. Guardrails before autonomy

Before giving the AI freedom to act, establish the rules it must follow. This is not micromanagement — it's the opposite. By investing in constraints upfront (coding guidelines, TDD workflow, PR process, naming conventions), you can later delegate with minimal supervision because the AI operates within safe boundaries.

**Example:**
Before any feature work begins, the groundwork is laid:
- "Please have a look at the README.md and the files loaded according to claude.md. Is there an explicit mention of TDD approach?"
- "Please add to our readme guideline that we should not define private functions or methods unless it is really a technical detail"
- "Also create a CODING_STYLE.md and a CLAUDE.md file in this directory"
- "Please add in README that new features are first merged against development and then development is merged to master"

Once these exist, later sessions can say "follow the instructions in the coding style and readme docs" (30 tokens) instead of re-explaining the entire workflow (300 tokens). The guardrails also apply to other users and other AI sessions — they're a one-time investment with compounding returns.

**The principle:**
Autonomy without guardrails is dangerous. Guardrails without autonomy is micromanagement. The most effective pattern is: invest heavily in rules once, then delegate freely.

---

## 6. Explicit action gates

Create deliberate pause points where the AI must report before executing. This prevents costly mistakes in high-stakes operations.

**Example:**
- "Only study, do not write code for now"
- "Don't act, only answer"
- "You will wait for my green light before merging anything"
- "Don't forget to git commit at the key step of your process"
- "I won't be answering for several hours, and I want to be back with the repository ready to review"

These gates create a two-phase workflow: *think, then act* — with a human checkpoint in between. The AI explores, proposes, analyzes — then waits. The human reviews, adjusts, approves — then the AI executes.

**When to gate:**
- Before any production change (deployment, database modification, model disabling)
- Before irreversible actions (merging PRs, deleting data)
- When the problem is not yet well understood ("read first, then we decide")
- When you're stepping away and want to review before the AI acts further

**The anti-pattern:**
Giving the AI a long chain of actions with no checkpoints: "analyze, implement, test, merge, deploy." If step 2 goes wrong, steps 3-5 compound the error.

---

## 7. Human judgment at decision points

Use the AI for data gathering, analysis, and execution — but keep the decisions for yourself. The AI computes; you decide.

**Example:**
The pattern repeats consistently:
- "Show me the benchmark results" → *AI produces table* → "Service B is clearly degraded, take it out of the load balancer"
- "Show me the table again without the decommissioned services"
- "Are you sure about 0.2ms average latency? That seems impossibly low for a cross-region call"
- "How can the Q1 numbers include the March outage if the window ends February 28?"

The user never says "decide which services to keep." They say "show me the data" and then make the call. When the AI presents numbers that don't make sense, the user challenges them — because the user has domain intuition that the AI lacks.

**Why it matters:**
The AI can process data faster, but it doesn't know what "unlikely" looks like in your domain. A 0.2ms cross-region latency might be mathematically valid but practically impossible. Only you know that. Keeping decision authority means the AI's mistakes get caught at review, not at deployment.

**How to apply:**
Ask for data, not decisions. Ask for options, not recommendations. When the AI presents results, apply your domain sense before acting. If something looks off, challenge it — the AI can be confidently wrong.

---

## 8. Challenge the AI's reasoning

Don't accept AI outputs at face value. Stress-test the logic, especially conclusions and causal claims.

**Example:**
- "You're concluding the new cache layer caused the slowdown, but you only tested one configuration. How would you know it's the cache and not the serialization format?" The AI concluded from N=1. The user caught the logical flaw.
- "Don't say 'further investigation needed for the cache layer' specifically. We could say the same about the thread pool or the connection timeout. The conclusion should be generic." The AI over-specified a conclusion that should have been generic.
- "NaN is not a valid value here. We have data for every region. If you're getting NaN, your query is wrong." The AI produced missing values instead of recognizing a bug in its own computation.
- "I never asked you to switch from the test window to the full history. This is wrong." The AI silently changed a parameter.

**Why it matters:**
The AI is fluent and confident. It will present wrong conclusions with the same tone as correct ones. The only defense is your own domain knowledge and critical thinking. Treating AI output as a draft to be reviewed, not a finished product, catches errors before they become actions.

**How to apply:**
When the AI presents a conclusion, ask yourself: would I accept this reasoning from a junior engineer? If not, push back. Pay special attention to causal claims ("X caused Y"), edge cases in data, and silent assumption changes.
