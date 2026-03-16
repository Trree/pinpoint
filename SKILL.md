---
name: pinpoint
description: "Research entry router. Frames the request through a 3-stage divergence pipeline, pauses for user confirmation, converges to one brief, then routes to deep-research, light-answer, needs-confirmation, or skill-route."
---

# Pinpoint

Research entry router. Analyze the request, converge on one brief, decide the downstream route. Do not answer the question.

## Trigger

Invoke when: broad or vague topic, mixed framings, claim needing framing before research, or any request needing routing between deep research / lightweight answer / confirmation / another skill.

Do not invoke when: simple factual lookup, direct writing request with no routing need.

## Pipeline

1. `Trend Scan`
2. `Anti-Consensus Scan`
3. `Perspective Expansion`
4. `User Confirmation` ← mandatory stop
5. `Consolidated Brief`
6. `Routing Decision`

## Analysis Layer

`analysis_block` fields: `original_question`, `trend_signals`, `anti_consensus_angles`, `perspective_lenses`, `user_confirmation`, `consolidated_brief`

### Trend Scan

Identify directional changes, emerging signals, timing windows, or shifting constraints. Outputs are framing hypotheses, not findings. Max 5 items. Use `none identified` if weak.

### Anti-Consensus Scan

Surface assumptions that may be weakly challenged or underexplored. Outputs are hypotheses or tensions, not conclusions. Max 4 items. Use `none identified` if weak.

### Perspective Expansion

Generate 3 framing lenses. Use exactly: `Lens 1`, `Lens 2`, `Lens 3`. Each populated lens must materially change the framing. Unused slots: `not applicable`. No paraphrase-only variation.

### User Confirmation

Mandatory pause. Present a compact summary of `trend_signals`, `anti_consensus_angles`, and `perspective_lenses`. Ask one focused question: which direction, angle, or lens to converge on. Do not proceed to `Consolidated Brief` until the user responds. If user says "continue" or gives no adjustment, treat the default recommended lens as confirmed. If user redirects, update framing before converging.

### Consolidated Brief

Mandatory convergence point. Required fields: `research_question`, `research_goal`, `boundaries`, `out_of_scope`, `key_terms`, `framing_rationale`

`boundaries` shape:

```json
{
  "subject": "string",
  "time_range": "string",
  "geography_or_context": "string",
  "evaluation_criteria": ["string"],
  "out_of_scope": ["string"]
}
```

Rules: exactly one recommended framing; alternatives only as discarded; if multiple framings remain equally live, do not advance past confirmation; `out_of_scope` must match `boundaries.out_of_scope`.

## Routing Layer

### Routing Decision

Required fields: `target`, `reason`, `confidence`, `needs_user_confirmation`

Invariant: `needs_user_confirmation = true` iff `target = needs-confirmation`; false for all other targets.

### Targets

Priority order — use first match:

1. `needs-confirmation` — unresolved ambiguity, missing key boundaries, or competing framings remain
2. `deep-research` — brief is converged and task warrants multi-source synthesis or report-style analysis
3. `skill-route` — brief is converged and a specific skill is a better match than deep-research
4. `light-answer` — question is clear and does not justify deep-research or a specific skill

`light-answer` is a dispatch contract for an outer agent; Pinpoint does not answer directly. If no outer dispatch exists, the outer agent may answer using the emitted payload.

## Deterministic Defaults

**Time normalization:** comparisons/evaluations without explicit range → `through YYYY-MM-DD` anchored to request date. Trend/landscape without explicit range → trailing 24-month range anchored to request date.

**Mode for `deep-research`:** `ultradeep` if explicitly requested; `deep` for high-stakes or rigorous requests; `quick` for exploratory-but-substantial; otherwise `standard`.

**Context inference:** infer geography/context only when low-risk; otherwise `needs-confirmation`.

**Multi-part requests:** keep together only if one deliverable and one shared research question; otherwise `needs-confirmation`.

## Route Payload Contract

Route shell:

```json
{
  "route_version": "2.0",
  "target": "deep-research | light-answer | needs-confirmation | skill-route",
  "payload": {}
}
```

Outer agent reads `target`, then parses only the relevant payload. When `target = deep-research`, user-facing output must be the legacy top-level `handoff_ready` JSON shape — no dual-format ambiguity.

### Payload: `deep-research`

Required fields: `handoff_version`, `terminal_state`, `target_skill`, `routing_reason`, `research_question`, `recommended_mode`, `research_goal`, `boundaries`, `assumptions`, `key_terms`, `success_criteria`, `next_action`

Constants (preserve exactly): `handoff_version: "1.1"`, `terminal_state: "handoff_ready"`, `target_skill: "deep-research"`, `next_action: "Invoke deep-research with this payload"`

Derive from `consolidated_brief`, not raw upstream signals. Emit as legacy top-level JSON directly.

### Payload: `light-answer`

Required fields: `question`, `answer_goal`, `constraints`, `suggested_depth`, `key_terms`

### Payload: `needs-confirmation`

Required fields: `recommended_framing`, `alternative_framings`, `missing_boundaries`, `next_user_action`

### Payload: `skill-route`

Required fields: `skill_id`, `routing_reason`, `skill_input`

`skill_id` must match the canonical installed skill name. `skill_input` envelope:

```json
{ "prompt": "string", "context": {}, "constraints": [] }
```

Outer agent must verify skill availability before dispatch. If unavailable: fall back to `light-answer` if still answerable; otherwise `needs-confirmation`.

## Output

Two passes. Do not execute downstream skills, gather evidence, or generate content drafts from inside Pinpoint.

**Pass 1 — divergence** (after Perspective Expansion): emit `divergence-confirmation`, stop, wait for user.

**Pass 2 — convergence** (after user responds): emit route payload, stop.

### divergence-confirmation

Emit: Trend Signals (max 5), Anti-Consensus Angles (max 4), Perspective Lenses (all 3 slots), one focused convergence question. Stop — do not emit routing or consolidated brief.

### needs-confirmation

Readable brief containing: Original Question, Trend Signals, Anti-Consensus Angles, Perspective Lenses, Consolidated Brief, Routing Decision, Recommended Framing, Alternative Framings, Next User Action. Then one fenced `json` block with the canonical route shell.

### deep-research

Optional one-line lead-in, then legacy top-level handoff JSON only. Do not emit readable analysis sections.

### light-answer / skill-route

Short routing summary, then one fenced `json` block with the canonical route shell.
