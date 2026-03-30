# Lecture 12. Why Every Session Must Leave a Clean State

## Question

Why must each agent session end in a clean, restartable state?

## Why It Matters

Long-running development systems degrade when sessions leave ambiguous progress,
stale artifacts, or unresolved structural violations. OpenAI and Anthropic both
indicate that reliability over time depends on operational discipline, not only
successful single runs.

## Core Concepts

- Clean state means the next session can start without manual repair.
- Benchmark slices are needed to detect harness drift over time.
- Cleanup loops reduce entropy in code, docs, and quality signals.
- Maintenance is part of normal harness operation, not emergency work.

## Detailed Explanation

OpenAI’s harness framing highlights repeatability: a workflow should continue to
produce reliable outcomes as tasks and contexts change. This requires measurable
checks beyond one successful demonstration. Benchmark comparisons help isolate
whether a harness change improved completion rate, reduced retries, or caught
more defects before review.

Anthropic’s long-running agent observations show why clean session exits are
critical. When unfinished work, stale guidance, or weak boundaries accumulate,
later sessions spend effort on state repair instead of forward progress. A clean
exit policy therefore includes explicit recording of progress, removal of stale
artifacts, and confirmation that the standard startup path still works.

Operationally, clean state and measurement reinforce each other:

1. Cleanup reduces noise.
2. Lower noise improves benchmark signal quality.
3. Clear benchmarks identify where cleanup policy should be strengthened.

## Examples and Artifacts

- [`code/benchmark-comparison-template.md`](./code/benchmark-comparison-template.md):
  template for comparing harness variants.
- [`code/cleanup-loop.md`](./code/cleanup-loop.md): recurring maintenance cycle
  for entropy reduction.
- [`code/README.md`](./code/README.md): index of lecture artifacts.

## Readings

Primary:
- OpenAI: *Harness engineering: leveraging Codex in an agent-first world*
- Anthropic: *Effective harnesses for long-running agents*

Secondary:
- Thoughtworks: *Harness Engineering*

## Exercises

1. Run a fixed task set on two harness variants and complete
   [`code/benchmark-comparison-template.md`](./code/benchmark-comparison-template.md).
   Identify which harness changed both outcome quality and operational cost.
2. Adopt [`code/cleanup-loop.md`](./code/cleanup-loop.md) for one week and track
   which recurring issues disappear versus persist.
3. Draft a clean-exit checklist for your repository. Include startup validation,
   progress recording, and stale-artifact removal criteria.
