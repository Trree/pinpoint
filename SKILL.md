---
name: pinpoint
description: "Expand broad, vague, or mixed questions into stronger research starting points before deeper analysis. Pinpoint first runs trend capture, anti-consensus exploration, and multi-perspective generation, then converges into one consolidated research brief. Default behavior: if the brief still needs framing, return the structured brief and stop; if it is already researchable and suitable for deep-research, emit the structured handoff payload for an outer agent to invoke deep-research."
---

# Pinpoint

Use this skill as a general research entrypoint.

Your job is not to answer the original question. Your job is to expand useful perspectives, converge on one research brief, and then decide whether the request is ready for research handoff.

This skill has exactly two terminal states:

- `needs_framing`: the request still needs narrowing, clarification, or a lighter non-deep-research next step
- `handoff_ready`: the question is already researchable and also appropriate for `deep-research`

## Trigger Rules

Invoke this skill when the user gives:

- a broad topic that needs narrowing
- a vague research question
- a mixed question that contains multiple subproblems
- a claim, thesis, or hypothesis that needs to be framed for research
- an explicit request to define, frame, narrow, or decompose a research question before analysis

Do not invoke this skill when the user gives:

- a simple factual lookup
- a clearly scoped data retrieval request
- a writing request such as an article, report, or newsletter draft
- a question that is already narrow enough for direct research

If the request is already researchable and appropriate for `deep-research`:

- say so briefly
- do not force candidate rewrites
- emit the structured `handoff_ready` payload
- do not do evidence gathering yourself

## Pipeline

Run the request through this sequence:

1. `Trend Scan`
2. `Anti-Consensus Scan`
3. `Perspective Expansion`
4. `Consolidated Research Brief`
5. `Researchability Check`

The first three stages are inputs to the brief. They are not independent terminal outputs.

### Trend Scan

Purpose:

- identify meaningful directional changes, emerging signals, timing windows, or shifting constraints

Policy:

- treat outputs as framing hypotheses, not asserted findings
- keep them concise and decision-relevant
- if no meaningful trend signals are identifiable, emit `none identified`

Output constraints:

- 3 to 5 trend signals maximum

### Anti-Consensus Scan

Purpose:

- surface mainstream assumptions and identify angles that may be underexplored, weakly challenged, or misread

Policy:

- treat outputs as framing hypotheses or tensions, not final claims
- if no credible anti-consensus angle is available, emit `none identified`

Output constraints:

- 2 to 4 anti-consensus angles maximum

### Perspective Expansion

Purpose:

- generate multiple useful lenses before convergence

Lens examples:

- market
- technology
- user
- policy
- competition
- distribution
- organizational capability

Policy:

- use exactly 3 labeled lens slots: `Lens 1`, `Lens 2`, `Lens 3`
- each populated lens must materially change how the research would be framed
- if the topic cannot support all 3 meaningful lenses, fill unused slots with `not applicable`
- avoid redundant rewordings

### Consolidated Research Brief

This is the mandatory convergence point.

Hard rules:

- always converge into one `Consolidated Research Brief`
- never stop at a raw list of insights, trend signals, or perspective lenses
- never hand off while multiple competing research questions remain unresolved

The brief must contain:

- chosen research question
- research goal
- key boundaries
- explicit out-of-scope area
- core terms
- why this framing is preferable to the discarded alternatives

## Readiness Criteria

Mark the question as `handoff_ready` only when all of the following are true:

- the consolidated brief expresses one clear research subject
- the consolidated brief expresses one clear research question
- the time range is explicit or safely inferable
- the geography or context is explicit or safely inferable
- the task type is clear, such as compare, evaluate, trend, feasibility, landscape, chronology, or review
- the task is substantial enough for `deep-research`, meaning it needs multi-source synthesis, comparison, verification, or report-style analysis rather than a simple lookup
- either:
  - at least one evaluation criterion is explicit or safely inferable for evaluative tasks, or
  - the descriptive task has a clear organizing lens, such as trend, landscape, chronology, or mechanism
- the request is not mixing multiple unrelated research problems

Otherwise mark it as `needs_framing`.

Operational defaults:

- comparisons or evaluations without a time range default to an absolute `through YYYY-MM-DD` range anchored to the request date
- trend or landscape requests without a time range default to an absolute trailing 24-month range anchored to the request date
- geography or context may be inferred only when the broader default is low-risk; otherwise keep the question in `needs_framing`
- related multi-part requests stay together only when they support one deliverable and one shared research question

Mode selection defaults:

- explicit user request for `ultradeep` wins over all other mode rules
- otherwise use `deep` for high-stakes, decision-shaping, or explicitly rigorous requests
- otherwise use `quick` for exploratory but still researchable requests
- otherwise use `standard`
- `ultradeep` is only for maximum-comprehensiveness requests explicitly signaled by the user

## Workflow

Run these steps in order:

1. Preserve the user's original question.
2. Run `Trend Scan`.
3. Run `Anti-Consensus Scan`.
4. Run `Perspective Expansion`.
5. Infer the likely research goal behind the question when the intent is clear.
6. If intent is ambiguous, present alternatives as assumptions rather than silently choosing one.
7. Build one `Consolidated Research Brief` with:
   - object or subject
   - time range
   - geography or context
   - evaluation criteria
   - what is out of scope
8. If the brief is still ambiguous or too broad, rewrite it into 2 to 3 candidate research directions.
9. Diagnose whether the brief is `needs_framing` or `handoff_ready`.
10. If the terminal state is `needs_framing`, produce the structured brief and stop.
11. If the terminal state is `handoff_ready`, emit the structured handoff payload for `deep-research`.
12. Stop.

## Assumption Policy

- If intent is clear, infer the likely research goal and label it as inferred.
- If intent is ambiguous, present alternatives explicitly as assumptions.
- Ask a clarification question only when the ambiguity would materially change the research direction.
- Do not recommend a rewrite unless the original question is genuinely too broad, too vague, or mixed across multiple problems.

## Output Shape

Use one of these output shapes.

### Output Shape: `needs_framing`

Use a semi-structured format with this compact required backbone:

- `Original Question`
- `Trend Signals`
- `Anti-Consensus Angles`
- `Perspective Lenses`
- `Consolidated Research Brief`
- `Researchability Check`
- `Terminal State`
- `Why It Needs Narrowing`
- `Recommended Framing`
- `Next User Action`
- `Stop / Continue`

Add these sections when they materially help:

- `Problem Diagnosis`
- `Research Goal`
- `Key Terms And Boundaries`
- `Candidate Research Versions`
- `Research Brief`

If the request is clear but too lightweight for `deep-research`, explain that directly inside `Researchability Check` and use `Next User Action` to recommend a lighter next step instead of asking for more narrowing.

### Output Shape: `handoff_ready`

Use `Trend Scan`, `Anti-Consensus Scan`, `Perspective Expansion`, and `Consolidated Research Brief` as internal reasoning inputs, but do not emit them if doing so would break the outer-agent parsing contract.

Emit a machine-readable JSON block and nothing else except an optional one-line lead-in.

Required schema:

```json
{
  "handoff_version": "1.1",
  "terminal_state": "handoff_ready",
  "target_skill": "deep-research",
  "routing_reason": "Question is already researchable and requires deep-research style synthesis",
  "research_question": "string",
  "recommended_mode": "quick | standard | deep | ultradeep",
  "research_goal": "string",
  "boundaries": {
    "subject": "string",
    "time_range": "string",
    "geography_or_context": "string",
    "evaluation_criteria": ["string"],
    "out_of_scope": ["string"]
  },
  "assumptions": ["string"],
  "key_terms": ["string"],
  "success_criteria": ["string"],
  "next_action": "Invoke deep-research with this payload"
}
```

When producing `handoff_ready`:

- include all top-level keys
- prefer explicit values over vague placeholders
- infer defaults only when the inference is low-risk
- keep the payload stable so an outer agent can parse it reliably
- normalize inferred time ranges into absolute strings such as `through 2026-03-16` or `2024-03-16 to 2026-03-16`
- use only these mode values: `quick`, `standard`, `deep`, `ultradeep`
- use arrays for list fields even when empty
- use `[]` for `evaluation_criteria` only on descriptive tasks such as trend, landscape, or chronology
- do not use `null`; if a required value cannot be filled safely, do not emit `handoff_ready`
- do not include source lists, evidence plans, or final conclusions
- derive the payload from the `Consolidated Research Brief`, not directly from raw trend or lens outputs

## Research Brief

When you include a research brief, keep it short and practical. Summarize:

- the chosen research question
- the research goal
- the boundaries
- the key terms that matter for this round
- what is explicitly out of scope
- why this framing beats the discarded alternatives
- a one-line next research action after user confirmation

## Quality Bar

The goal is not to sound smart. The goal is to make the question researchable.

Prefer:

- narrower scope
- converged framing after useful perspective expansion
- explicit boundaries
- testable wording
- direct research utility

Avoid:

- answering the full question too early
- staying in a divergent insight list without converging
- keeping the wording broad but impressive
- over-defining irrelevant terms
- silently replacing the user's intent
- forcing one framing when multiple research directions are plausible

## Stop Rule

After the workflow, stop in one of two ways:

- if `needs_framing`, stop with a structured brief and the exact next user action
- if `handoff_ready`, stop after emitting the structured payload so an outer agent can invoke `deep-research`

Do not automatically proceed into:

- evidence gathering
- framework building
- counterarguments
- scenario analysis
- recursive follow-up research
- direct execution of `deep-research` from inside this skill

Stop boundary:

- allow at most a one-line next step outside the JSON payload
- do not include source lists
- do not include framework trees
- do not include evidence-gathering plans
- do not expand into subquestion research before confirmation

## References

Load these only if the user explicitly asks to continue beyond phase 1:

- `references/phase-2-framework.md` for framework building
- `references/phase-3-evidence.md` for evidence gathering
- `references/phase-4-counterarguments.md` for counterarguments and failure cases
- `references/phase-5-scenarios.md` for scenario analysis
- `references/phase-6-synthesis.md` for final synthesis and conclusion shaping
- `references/output-examples.md` for example outputs
