---
layout: post
title: "Effective AI Usage Patterns"
description: "Eight patterns for working with AI, drawn from over 500 real engineering prompts."
author: Jeremy Cochoy
---

I read over 500 prompts from engineers who use AI every day on production systems. Eight patterns came out of that reading. Each one describes how to split work with a tool that is fast and confident even when it is wrong.

---

## 1. Verify before you commit scope

Do not open with "analyze all the data and write me a report." Start small: "Do you see the data?" Then: "Who are the users?" Then: "Analyze them." Each step tests a capability before you commit more scope.

A conversation that follows this pattern:

1. "Can you access the production database on the staging server?"
2. "Yes, use ssh deploy@staging"
3. "I want you to compare the last 3 deployments and flag any performance regressions"
4. "Also include memory usage, not just latency"
5. "Write this up as a summary report"

Step 1 tests reach. Step 2 hands over the credential. The real task only arrives at step 3. Step 4 steers after the first output comes back, and step 5 commits to a deliverable once the content holds up.

Each step is a cheap checkpoint. A failure at step 1 costs you one line instead of a detailed analysis prompt.

Compare with the opposite approach: one long prompt that says "SSH to staging, query the database, compare the last 3 deployments across latency and memory, and write a formatted summary report." One wrong assumption (no SSH access, a different schema, five deployments instead of three) throws away the work.

---

## 2. Context is worth keeping

A conversation you keep alive for days is worth more than a fresh one. Corrections and domain details you shared earlier stay in the context window and shape later answers. That is what lets you steer with four-word prompts instead of a full brief.

One session about migrating a legacy API ran 12 days and more than 50 prompts. By day 10 the prompts were "Check the logs", "Yes, deploy it", "Rollback the auth service". Three or four words each, and each one precise, because both sides already knew which endpoints had moved and how rollback works here.

There is a trade-off. A fresh session costs you a long re-explanation but carries no stale context. An old session is cheap to steer but can drag along assumptions that no longer hold. Keep a session while the problem stays the same. Open a new one when the problem changes.

Do not close a session because you are stepping away. Come back to it tomorrow, and re-read the last exchange before you continue.

---

## 3. Change what the AI is trying to do

The instinct on a bad output is to point at it and say "fix this."

Here is a case. The AI produced a prompt analysis that judged short prompts as a weakness, saying they "would be useless as standalone documentation." There were two ways to answer:

- Correct the output: "Short prompts aren't a weakness, they're efficient." The AI adjusts that one judgment, then keeps evaluating prompts as if they were meant to be self-contained instructions.
- Correct the framing: "Prompts aren't supposed to be documentation. They give us a window on how the user works, and that is what we want to understand."

The second answer changed the whole analysis. The AI re-derived its conclusions from the corrected premise, including ones it had not produced yet.

Correcting an output changes that one answer, while correcting the premise changes everything that follows from it.

So before you retype the request, check which one is wrong: the answer, or the AI's idea of what you want. If it is the second, say what it should be aiming at.

---

## 4. Let it work while you are away

The AI can run a task in the background and keep checking on it while you do something else. Treat it as an operator that stays on duty.

Prompts of this kind:

- "Run the full regression suite. Check every 20 minutes. If a test fails, investigate but don't push any fix until I review."
- "Good morning. Did the overnight migration finish? Show me the error count."
- "Start the deployment. Monitor the health checks for 3 hours, and rollback if error rate exceeds 1%."

A working day then has a shape. A short burst of prompts to set the task up and steer it. Then hours, or a night, with no contact. Then a check-in, "How is it going?", and either another burst or a final "merge and deploy."

On a long task such as a training run or a deployment, do not sit and watch. Say what to monitor and what to do if it goes wrong, then leave.

---

## 5. Write the rules down before you delegate

Write the rules down first: coding guidelines, TDD workflow, PR process, naming conventions. Once they exist you can delegate with light supervision, because the AI has something to check itself against.

The setup prompts look like this:

- "Please have a look at the README.md and the files loaded according to claude.md. Is there an explicit mention of TDD approach?"
- "Please add to our readme guideline that we should not define private functions or methods unless it is a technical detail"
- "Also create a CODING_STYLE.md and a CLAUDE.md file in this directory"
- "Please add in README that new features are first merged against development and then development is merged to master"

After that, a later session only needs "follow the instructions in the coding style and readme docs" instead of a full description of the workflow. The same files apply to other people and to other AI sessions, so you write them once.

---

## 6. Explicit action gates

Create pause points where the AI must report before it executes.

- "Only study, do not write code for now"
- "Don't act, only answer"
- "You will wait for my green light before merging anything"
- "Don't forget to git commit at the key step of your process"
- "I won't be answering for several hours, and I want to be back with the repository ready to review"

A gate splits the work in two. The AI investigates and proposes, then stops. You review and approve, then it runs.

Gate before:

- Any production change (deployment, database modification, model disabling)
- Any irreversible action (merging PRs, deleting data)
- Any problem you do not yet understand ("read first, then we decide")
- Any period when you step away and want to review before the AI acts further

Without a gate you get a long chain of actions and no checkpoints: "analyze, implement, test, merge, deploy." If step 2 goes wrong, steps 3 to 5 build on the error.

---

## 7. Do not delegate your judgment

Your judgment is the highest-value thing you bring to the work. Hand over the data gathering and the analysis. Keep the thinking. Ask for numbers and for options, then make the call yourself.

- "Show me the benchmark results", then, after reading the table, "Service B is degraded, take it out of the load balancer"
- "Show me the table again without the decommissioned services"
- "Are you sure about 0.2ms average latency? That seems impossibly low for a cross-region call"
- "How can the Q1 numbers include the March outage if the window ends February 28?"

The AI processes data faster than you do, but it has no sense of what is unlikely in your domain. A 0.2ms cross-region latency is arithmetically fine and physically impossible. Your review is what catches that.

---

## 8. Challenge the AI's reasoning

The previous pattern protects your own thinking. This one audits the AI's. A model drifts into overconfidence easily, and it skips steps quietly. Check the conclusions and the causal claims first.

- "You're concluding the new cache layer caused the slowdown, but you only tested one configuration. How would you know it's the cache and not the serialization format?" The AI had concluded from N=1.
- "Don't say 'further investigation needed for the cache layer' specifically. We could say the same about the thread pool or the connection timeout. The conclusion should be generic." The AI had over-specified a conclusion.
- "NaN is not a valid value here. We have data for every region. If you're getting NaN, your query is wrong." The AI reported missing values instead of finding the bug in its own query.
- "I never asked you to switch from the test window to the full history. This is wrong." The AI had changed a parameter without saying so.

A wrong conclusion arrives in the same confident tone as a correct one. Read the output as a draft. Three things deserve a second look:

- Causal claims, in the form "X caused Y"
- Edge cases in the data
- Parameters that changed without a word
