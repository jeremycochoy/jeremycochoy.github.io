---
layout: page
title: "Claude Opus 4.7/4.8 on SWE-PRBench"
description: "Results from running SWE-PRBench on Claude Opus 4.7 (max effort) and Opus 4.8 (low / medium / max effort), judged by Claude Sonnet 4.6."
permalink: /swe-prbench/
---

We ran the [SWE-PRBench](https://arxiv.org/abs/2603.26130) benchmark (`eval_100` split, 100 PRs × 3 context configurations) on four agent configurations: Claude Opus 4.7 at max effort, and Claude Opus 4.8 at low, medium, and max effort. The judge model is Claude Sonnet 4.6.

## config_A (diff only)

| Agent             | Overall | Detection | Hallucination | F1    |
|-------------------|---------|-----------|---------------|-------|
| `opus_4_8_medium` | 0.479   | 0.665     | 0.082         | 0.517 |
| `opus_4_8_max`    | 0.467   | 0.642     | 0.072         | 0.496 |
| `opus_4_8_low`    | 0.466   | 0.633     | 0.080         | 0.500 |
| `opus_4_7_max`    | 0.452   | 0.696     | 0.068         | 0.447 |

## config_B (with file content)

| Agent             | Overall | Detection | Hallucination | F1    |
|-------------------|---------|-----------|---------------|-------|
| `opus_4_8_max`    | 0.416   | 0.564     | 0.059         | 0.446 |
| `opus_4_8_low`    | 0.414   | 0.570     | 0.058         | 0.455 |
| `opus_4_8_medium` | 0.410   | 0.549     | 0.048         | 0.435 |
| `opus_4_7_max`    | 0.394   | 0.613     | 0.055         | 0.395 |

## config_C (full context)

| Agent             | Overall | Detection | Hallucination | F1    |
|-------------------|---------|-----------|---------------|-------|
| `opus_4_8_low`    | 0.430   | 0.593     | 0.053         | 0.471 |
| `opus_4_8_max`    | 0.425   | 0.581     | 0.057         | 0.464 |
| `opus_4_8_medium` | 0.412   | 0.557     | 0.058         | 0.450 |
| `opus_4_7_max`    | 0.387   | 0.606     | 0.055         | 0.391 |

## Final scores (mean across A, B, C)

| Rank | Agent             | Overall |
|------|-------------------|---------|
| 1    | `opus_4_8_low`    | 0.437   |
| 2    | `opus_4_8_max`    | 0.436   |
| 3    | `opus_4_8_medium` | 0.434   |
| 4    | `opus_4_7_max`    | 0.411   |

---

[Source data and reproduction harness](https://github.com/jeremycochoy/swe-prbench/tree/add-claude-code-cli-eval)
