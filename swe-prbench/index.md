---
layout: page
title: "Claude Opus 4.7/4.8 on SWE-PRBench"
description: "Extending FoundryHQ's SWE-PRBench harness to drive the local claude CLI under an OAuth subscription, then running Opus 4.7 and Opus 4.8 across the low/medium/max effort tiers. Higher effort barely moves the score."
permalink: /swe-prbench/
---

[SWE-PRBench](https://arxiv.org/abs/2603.26130) is a benchmark of 350 human-annotated GitHub pull requests, designed to measure whether AI code reviewers actually catch the issues that human reviewers flagged. Unlike SWE-Bench (which measures whether a model can *write* a fix), SWE-PRBench measures whether a model can *evaluate* code as an expert reviewer would. The dataset, judge framework, and harness are public, but the published leaderboard doesn't include Claude Opus 4.7 or 4.8.

I extended the harness to fill that gap. This page is the result.

## What I added

The upstream harness routes every model call through HTTP — Anthropic SDK, OpenAI Chat Completions, Google Gemini. Anthropic API credits on the Opus tier add up fast for a multi-thousand-call sweep, so I added a `cli_claude` provider that shells out to the local `claude` CLI in headless mode:

```bash
claude -p --model claude-opus-4-8 --effort low \
       --tools "" --no-session-persistence --safe-mode \
       --disable-slash-commands --no-chrome \
       --system-prompt "<rubric>" --output-format text
```

Every call routes through the user's logged-in OAuth subscription. Same input prompt as the upstream harness, same one-shot semantics, same JSON output. Total experiment cost: zero API dollars (one Pro subscription, eight 5-hour usage windows). The full provider is ~80 LoC in `model_clients.py`.

The judge is **Claude Sonnet 4.6** instead of GPT-5.2. Sonnet 4.6 was the paper's cross-validator (κ=0.75 with GPT-5.2), so it's inside the paper's validated judge family — but absolute scores here are not drop-in comparable to the published leaderboard.

## Setup

Four agent configurations, all routed through `cli_claude`:

| Agent | Model | Effort |
|---|---|---|
| `opus_4_8_low` | claude-opus-4-8 | low |
| `opus_4_8_medium` | claude-opus-4-8 | medium |
| `opus_4_8_max` | claude-opus-4-8 | max |
| `opus_4_7_max` | claude-opus-4-7 | max |

Evaluated on the `eval_100` split: 100 PRs across 3 context configurations (A = diff only ~2k tokens, B = diff + file content ~2.2k, C = diff + full context ~2.5k). 1200 records total, zero parse failures.

## Headline results

| Rank | Agent | Overall | DR_A | Halluc_A | F1_A |
|------|-------|---------|------|----------|------|
| 1 | `opus_4_8_low` | 0.437 | 0.633 | 0.080 | 0.500 |
| 2 | `opus_4_8_max` | 0.436 | 0.642 | 0.072 | 0.496 |
| 3 | `opus_4_8_medium` | 0.434 | 0.665 | 0.082 | 0.517 |
| 4 | `opus_4_7_max` | 0.411 | 0.696 | 0.068 | 0.447 |

*Overall = mean across config_A/B/C. DR_A = detection rate on diff-only. Halluc_A = fabricated-comment rate on diff-only. F1_A = F1 on diff-only.*

Three things stood out.

## Finding 1: the reasoning-effort axis is essentially flat

The big surprise. Opus 4.8 at `low`, `medium`, and `max` effort lands within **0.003** of each other on overall_score. That's noise.

On config_A specifically, `medium` does win (0.479 vs 0.466 for `low`), but B and C flip the order again — there's no monotonic ramp. If reasoning effort were doing real work, you'd expect to see it in this kind of analytic task, where "think harder about whether this line is correct" should help. It doesn't.

My read: code review on a diff is dominated by reading comprehension and pattern recognition over a small context, not by deliberative reasoning. The model either spots the issue on its first pass or it doesn't — extra thinking tokens just produce more elaborate-but-not-more-accurate analyses. Worth running this on bigger contexts to see if the pattern holds.

If you're picking an effort tier for code review specifically: **use low**. You'll get the same score for a fraction of the latency.

## Finding 2: the best detector is the lowest-ranked agent

`opus_4_7_max` has the highest detection rate (0.696) AND the lowest hallucination rate (0.068) of all four configurations. By any normal measure of "is this a good reviewer," it's the best of the four. But it finishes last on overall_score.

Looking at the comment-count breakdown:

| Agent | Comments/PR | %CONFIRMED | %PLAUSIBLE | %FABRICATED |
|---|---|---|---|---|
| `opus_4_7_max` | **4.88** | 40.3% | 52.9% | 6.8% |
| `opus_4_8_medium` | 3.58 | 52.2% | 41.1% | 6.7% |
| `opus_4_8_max` | 3.61 | 51.2% | 41.6% | 7.2% |
| `opus_4_8_low` | 3.56 | 51.7% | 40.4% | 7.9% |

Opus 4.7 max writes ~37% more comments per PR. The extras are overwhelmingly **PLAUSIBLE** — credible code observations that the judge classifies as "no factual errors, but no human reviewer raised this point." The harness's precision metric (`confirmed / total_comments`) divides the denominator by every comment, so a thorough reviewer that catches everything humans flagged *and* adds extra observations gets punished for the extras.

This is a real benchmark artefact, not a model artefact. Opus 4.7 max is doing exactly what you'd want a senior reviewer to do — flag every credible issue, including some humans missed — and the metric reads it as noise. Worth keeping in mind when interpreting any benchmark that combines precision and recall into a single score.

## Finding 3: more context hurts detection

The paper's headline finding was that a structured 2K-token diff prompt outperforms a 2.5K full-context prompt across all 8 of their models. Same pattern here on the Claude Opus family:

| Agent | config_A | config_B | config_C | A → C drop |
|-------|----------|----------|----------|------------|
| `opus_4_7_max` | 0.696 | 0.620 | 0.621 | -7.5 pp |
| `opus_4_8_medium` | 0.665 | 0.549 | 0.566 | -9.9 pp |
| `opus_4_8_low` | 0.633 | 0.570 | 0.572 | -6.1 pp |
| `opus_4_8_max` | 0.642 | 0.564 | 0.554 | -8.8 pp |

Detection drops by 6–10pp going from diff-only to with-file-content. Going further to full context doesn't recover.

Mechanism: when the model has only the diff, all 4–6 of its budgeted comments land on changed lines. When it also has the surrounding file, some comments wander onto code that humans didn't review. Those land as PLAUSIBLE (or occasionally FABRICATED), not as CONFIRMED matches. Recall suffers because the comment budget is being spent on non-ground-truth observations.

Interestingly, hallucination drops *too* with more context (medium goes 8.2% → 4.7% → 3.5% across A/B/C). So extra context does make the model more factually careful — it just makes detection drop faster than hallucination falls.

## Reproducibility

Fork: [github.com/jeremycochoy/swe-prbench, branch `add-claude-code-cli-eval`](https://github.com/jeremycochoy/swe-prbench/tree/add-claude-code-cli-eval).

Everything is committed: the `cli_claude` provider, the per-agent YAML config, driver scripts (sequential and parallel balancers, usage-aware orchestration, contamination-aware resume), the final leaderboard CSV, and per-run `eval_report.json` files. The per-PR raw outputs aren't in git (too big), but I have them on disk and am happy to share if anyone wants to redo the κ analysis on a Claude-vs-Claude judge pair.

If you want to run this on your own subscription, you need: the harness fork, the dataset (`hf download foundry-ai/swe-prbench --repo-type dataset`), Python 3.10+, and a Claude Code install. Then:

```bash
bash claude_code_eval/scripts/run_experiment.sh smoke   # 3 PRs, sanity check
bash claude_code_eval/scripts/run_experiment.sh         # 4 agents × 100 PRs
```

It will absolutely consume your subscription's session windows. The orchestrator in the fork stops the runs at 60% session usage and auto-resumes after the 5-hour reset, so you can leave it running across multiple windows without burning through your weekly cap.

---

*Hat tip to Deepak Kumar for releasing the dataset, harness, and judge framework openly — that's the only reason an experiment like this is possible without rebuilding the ground truth from scratch.*
