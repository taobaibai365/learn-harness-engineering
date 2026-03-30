# Lecture 05. Why Long-Running Tasks Lose Continuity

## Question

Why do agent projects lose continuity across sessions, and what must a harness
persist to prevent that loss?

## Why It Matters

Long-running development is a sequence of handoffs. Each handoff can introduce
state loss, ambiguity, and duplicated effort. If continuity is weak, the agent
spends increasing time reconstructing context instead of shipping verified
behavior.

## Core Concepts

- Session boundary: a transition where a new run does not automatically inherit
  complete prior reasoning.
- Working context vs durable state: model context is transient; repository
  artifacts persist.
- Continuity artifact: an external record (file, commit, checklist, status
  entry) that supports reliable handoff.
- Reconstruction cost: time and tokens spent rediscovering project state.
- Drift: divergence between intended state and actual repository/runtime state.

## Detailed Explanation

Anthropic’s long-running harness work frames continuity as a systems problem.
Even strong models underperform when each run must infer history from partial
signals. The failure mode is not only "bad generation"; it is missing
infrastructure for handoff.

OpenAI’s harness engineering guidance reinforces the same principle from another
angle: the repository should be the operational system of record, not the chat
transcript. Continuity improves when each run leaves explicit, inspectable
artifacts that answer:

1. What is currently true?
2. What is failing or unfinished?
3. What is the next executable step?

Context summaries can reduce short-term rediscovery, but summaries alone are
not durable state. Long-running reliability requires artifacts that can be
versioned, diffed, and validated across sessions. In practice, this means the
harness must treat handoff quality as a formal requirement, not incidental
documentation.

## Examples and Artifacts

- See [`code/`](./code/README.md) for continuity failure and recovery examples.
- Typical continuity artifacts include a progress log, a feature-status file, a
  clean commit checkpoint, and a short verification record from the latest run.
- A useful handoff artifact states repository status, runtime status, blockers,
  and next action in directly testable terms.

## Readings

- Anthropic: Effective harnesses for long-running agents
- Anthropic: Harness design for long-running application development
- OpenAI: Harness engineering: leveraging Codex in an agent-first world

## Exercises

1. Run the same multi-step task in two sessions: once without continuity
   artifacts, and once with a progress log plus feature-status file. Compare
   reconstruction time.
2. Ask a fresh session to resume work using repository artifacts only (no prior
   conversation context). Record which details were recoverable and which were
   missing.
3. Design a minimal handoff template with exactly four fields. Validate whether
   a new session can safely continue work from that template.
