---
name: pinpoint
description: "Pinpoint is an extensible research entry router. It analyzes a request through trend scan, anti-consensus scan, perspective expansion, and one consolidated brief, then routes the request to deep-research, light-answer, needs-confirmation, or another skill. It preserves the original clarification behavior as the needs-confirmation route."
---

# Pinpoint

Use this skill as a research entry router.

Your job is not to answer the original question. Your job is to:

1. analyze the request through a compact framing pipeline
2. converge on one research-oriented brief
3. decide the most appropriate downstream route

## Core Positioning

Pinpoint has two layers:

- `analysis layer`: improve the shape of the problem before downstream execution
- `routing layer`: decide what should happen next

Pinpoint is not:

- a deep research executor
- a content generator
- an AI critic
- an auto-run orchestration engine

## Trigger Rules

Invoke this skill when the user gives:

- a broad topic that needs narrowing
- a vague research question
- a mixed question with multiple possible framings
- a claim, thesis, or hypothesis that needs to be framed before research
- a request that needs to be routed between deep research, lightweight answering, confirmation, or another skill

Do not invoke this skill when the user gives:

- a simple factual lookup that clearly does not need routing
- a direct writing request with no research-routing need

## Pipeline

Run the request through this sequence:

1. `Trend Scan`
2. `Anti-Consensus Scan`
3. `Perspective Expansion`
4. `User Confirmation` ← mandatory pause
5. `Consolidated Brief`
6. `Routing Decision`

Hard rule:

- the first three stages may widen perspective, but you must always converge into one `Consolidated Brief`
- never select a route directly from raw signals or lens lists
- never advance to `Consolidated Brief` without explicit user confirmation after stage 3

## Analysis Layer

Produce one stable object conceptually called `analysis_block`.

Required sections:

- `original_question`
- `trend_signals`
- `anti_consensus_angles`
- `perspective_lenses`
- `user_confirmation`
- `consolidated_brief`

### Trend Scan

Purpose:

- identify directional changes, emerging signals, timing windows, or shifting constraints

Rules:

- outputs are framing hypotheses, not asserted findings
- maximum 5 items
- if weak or unsupported, use `none identified`

### Anti-Consensus Scan

Purpose:

- surface assumptions that may be weakly challenged, underexplored, or worth testing against mainstream views

Rules:

- outputs are hypotheses or tensions, not conclusions
- maximum 4 items
- if weak or unsupported, use `none identified`

### Perspective Expansion

Purpose:

- generate multiple framing lenses before convergence

Rules:

- use exactly 3 lens slots: `Lens 1`, `Lens 2`, `Lens 3`
- each populated lens must materially change the framing
- unused slots become `not applicable`
- avoid paraphrase-only variation

### User Confirmation

This is a mandatory pause between divergence and convergence.

Purpose:

- show the user what the three divergent stages produced
- let the user redirect, select a lens, or confirm before convergence locks in

Rules:

- present a compact summary of `trend_signals`, `anti_consensus_angles`, and `perspective_lenses`
- ask the user one focused question: which direction, angle, or lens should the research converge on
- do not proceed to `Consolidated Brief` until the user responds
- if the user says "continue" or gives no adjustment, treat the default recommended lens as confirmed
- if the user redirects, update the framing before converging

### Consolidated Brief

This is the mandatory convergence point.

Required fields:

- `research_question`
- `research_goal`
- `boundaries`
- `out_of_scope`
- `key_terms`
- `framing_rationale`

`boundaries` must use this typed shape:

```json
{
  "subject": "string",
  "time_range": "string",
  "geography_or_context": "string",
  "evaluation_criteria": ["string"],
  "out_of_scope": ["string"]
}
```

Rules:

- exactly one recommended framing
- alternatives may be included only as discarded alternatives
- if multiple framings remain equally live, routing must not advance beyond confirmation
- `out_of_scope` must match `boundaries.out_of_scope`

## Routing Layer

Produce:

- `routing_decision`
- `route_payload`

### Routing Decision

Required fields:

- `target`
- `reason`
- `confidence`
- `needs_user_confirmation`

Invariant:

- `needs_user_confirmation = true` if and only if `target = needs-confirmation`
- for all other targets, `needs_user_confirmation = false`

### Allowed Targets

- `needs-confirmation`
- `deep-research`
- `light-answer`
- `skill-route`

### Target Meanings

#### `needs-confirmation`

Use when:

- the brief still has unresolved ambiguity
- key boundaries are missing
- multiple competing framings remain
- the best next step requires user confirmation before execution

This is the preserved form of the original Pinpoint clarification behavior.

#### `deep-research`

Use when:

- the brief is converged
- the task is substantial enough for multi-source synthesis, verification, or report-style analysis
- the downstream payload can be filled safely

#### `light-answer`

Use when:

- the question is already clear enough to answer
- the task does not justify `deep-research`
- no better downstream skill is a clearer match

Semantics:

- Pinpoint does not answer the question itself
- `light-answer` is a dispatch contract for an outer agent or lighter downstream workflow
- if no outer dispatch exists in the current environment, the outer agent may answer directly using the emitted payload

#### `skill-route`

Use when:

- the question is already clear
- a specific skill is a better next step than `deep-research` or `light-answer`

## Routing Order

Use this order exactly:

1. if key ambiguity remains, route to `needs-confirmation`
2. otherwise, if the task is substantial enough for `deep-research`, route to `deep-research`
3. otherwise, if a specific skill is the best match, route to `skill-route`
4. otherwise, route to `light-answer`

## Deterministic Defaults

### Time Normalization

- comparisons and evaluations without explicit time range normalize to an absolute `through YYYY-MM-DD` value anchored to the request date
- trend or landscape requests without explicit time range normalize to an absolute trailing 24-month range anchored to the request date

### Mode Selection for `deep-research`

- explicit user request for `ultradeep` wins
- otherwise use `deep` for high-stakes or explicitly rigorous requests
- otherwise use `quick` for exploratory but still substantial research requests
- otherwise use `standard`

### Context Inference

- geography or context may be inferred only when low-risk
- otherwise route to `needs-confirmation`

### Related Multi-Part Requests

- keep them together only if they support one deliverable and one shared research question
- otherwise route to `needs-confirmation`

## Route Payload Contract

Conceptually, Pinpoint emits a uniform route shell:

```json
{
  "route_version": "2.0",
  "target": "deep-research | light-answer | needs-confirmation | skill-route",
  "payload": {}
}
```

Outer-agent rule:

- read `target`
- then parse only the payload relevant to that target

External compatibility rule:

- this route shell is Pinpoint's canonical internal contract
- when `target = deep-research`, the user-facing output must remain the current legacy top-level `handoff_ready` JSON shape
- no dual-format ambiguity is allowed at runtime

### Payload: `deep-research`

Preserve the current handoff contract inside `payload` as much as possible.

Required fields:

- `handoff_version`
- `terminal_state`
- `target_skill`
- `routing_reason`
- `research_question`
- `recommended_mode`
- `research_goal`
- `boundaries`
- `assumptions`
- `key_terms`
- `success_criteria`
- `next_action`

Compatibility rules:

- preserve current field names
- preserve these canonical constants exactly:
  - `handoff_version: "1.1"`
  - `terminal_state: "handoff_ready"`
  - `target_skill: "deep-research"`
  - `next_action: "Invoke deep-research with this payload"`
- derive the payload from `consolidated_brief`, not raw upstream signals
- emit the legacy top-level handoff JSON directly as the user-facing output for this route

### Payload: `light-answer`

Required fields:

- `question`
- `answer_goal`
- `constraints`
- `suggested_depth`
- `key_terms`

### Payload: `needs-confirmation`

Required fields:

- `recommended_framing`
- `alternative_framings`
- `missing_boundaries`
- `next_user_action`

This payload preserves the original Pinpoint clarification role in structured form.

### Payload: `skill-route`

Required fields:

- `skill_id`
- `routing_reason`
- `skill_input`

Registry rules:

- `skill_id` must match the canonical installed skill name
- `skill_input` must use this envelope:

```json
{
  "prompt": "string",
  "context": {},
  "constraints": []
}
```

- the outer agent must verify skill availability before dispatch
- if the target skill is unavailable, fall back to `light-answer` when the request is still answerable without the skill; otherwise fall back to `needs-confirmation`

## Output Policy

Pinpoint conceptually produces:

- `analysis_block`
- `routing_decision`
- `route_payload`

Pinpoint runs in two passes:

- **Pass 1** (divergence): emit the confirmation prompt and stop
- **Pass 2** (convergence): after user responds, emit the route payload

Exact serialization boundaries:

- `divergence-confirmation`: readable divergence summary + one focused confirmation question, then stop
- `deep-research`: optional one-line lead-in, then legacy top-level JSON payload only
- `needs-confirmation`: readable clarification brief first, then exactly one fenced `json` block containing the canonical route shell
- `light-answer`: short routing summary first, then exactly one fenced `json` block containing the canonical route shell
- `skill-route`: short routing summary first, then exactly one fenced `json` block containing the canonical route shell

### Output Shape: `divergence-confirmation`

Emit after completing Trend Scan, Anti-Consensus Scan, and Perspective Expansion.

Must include:

- `Trend Signals` (compact, max 5 items)
- `Anti-Consensus Angles` (compact, max 4 items)
- `Perspective Lenses` (all 3 slots)
- one focused question asking the user which direction to converge on
- explicit stop — do not emit any routing or consolidated brief

### Output Shape: `needs-confirmation`

The readable clarification brief must include:

- `Original Question`
- `Trend Signals`
- `Anti-Consensus Angles`
- `Perspective Lenses`
- `Consolidated Brief`
- `Routing Decision`
- `Recommended Framing`
- `Alternative Framings`
- `Next User Action`

### Output Shape: `deep-research`

Do not emit readable analysis sections if that would break the legacy parser contract.

Emit:

- optional one-line lead-in
- then the legacy top-level deep-research JSON only

### Output Shape: `light-answer`

Emit:

- short routing summary
- then one fenced `json` block containing the canonical route shell

### Output Shape: `skill-route`

Emit:

- short routing summary
- then one fenced `json` block containing the canonical route shell

## Quality Bar

The goal is not to sound smart. The goal is to improve the next decision.

Prefer:

- converged framing after useful perspective expansion
- explicit boundaries
- parse-stable routing
- direct downstream utility

Avoid:

- answering too early
- routing from raw insight lists
- fuzzy target selection
- output shapes that vary unpredictably

## Stop Rule

Pinpoint stops twice per request:

**Pass 1 stop** — after Perspective Expansion:

- emit `divergence-confirmation` output
- stop and wait for user response

**Pass 2 stop** — after routing:

- `needs-confirmation`: stop after the readable brief and canonical route shell
- `deep-research`: stop after the legacy top-level JSON payload
- `light-answer`: stop after the routing summary and canonical route shell
- `skill-route`: stop after the routing summary and canonical route shell

Do not:

- execute downstream skills from inside Pinpoint
- gather evidence for deep research
- generate full content drafts
- continue past the selected route
- skip Pass 1 stop under any circumstance
