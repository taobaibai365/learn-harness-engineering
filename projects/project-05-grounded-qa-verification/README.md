# Project 05. Evaluator Loops and Three-Role Upgrades

## Objective

Measure how role separation (generator and evaluator, then planner, generator,
and evaluator) changes implementation quality on a substantial product upgrade.

## Learning Goals

- Separate generation from evaluation to improve error detection.
- Apply a rubric-driven review loop instead of subjective quality checks.
- Compare coordination overhead versus output quality across harness designs.

## Task

Choose one substantial upgrade and keep it fixed across all runs:

- Multi-turn Q&A history.
- Citation panel redesign.
- Document collections and filtering.
- Indexing recovery with richer Q&A behavior.

Run three harness variants on the same upgrade scope.

## Baseline Harness Setup

Run 1 uses a single-role harness:

- One agent plans, implements, and self-judges.
- No external evaluator rubric is required.

## Improved Harness Setup

Run 2 and Run 3 use upgraded role separation:

- Run 2: `generator + evaluator`.
- Run 3: `planner + generator + evaluator`.
- Evaluator must score output with the same rubric in both improved runs.

## Procedure

1. Select upgrade scope and define acceptance criteria before coding.
2. Start three branches from the same commit:
   `p05-single`, `p05-gen-eval`, `p05-plan-gen-eval`.
3. Run each branch with Codex or Claude Code using comparable model and budget.
4. Use the same evaluator rubric for all three final outputs.
5. In improved runs, require at least one revision round after evaluator
   feedback.
6. Verify behavior with runnable evidence and record unresolved defects.
7. Compare quality and effort across the three harness variants.

## What to Measure

- Scope definition quality before coding starts.
- Number and severity of defects caught before final handoff.
- Rubric score by category (correctness, reliability, maintainability, UX).
- Rework required after review.
- Final robustness of the upgraded product slice.

## Deliverables

- Single-role run record (prompt, transcript/log, runnable evidence).
- Generator-evaluator run record (including rubric and revision notes).
- Planner-generator-evaluator run record (including rubric and revisions).
- One comparison note summarizing quality gains and coordination costs.
