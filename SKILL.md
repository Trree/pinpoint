---
name: pinpoint
description: "Turn broad, vague, or mixed questions into clear research problems before deeper analysis. Use when the user wants to research, analyze, compare, evaluate, or think through a topic but the question still needs framing, narrowing, decomposition, or boundary-setting. Also use when the user explicitly asks to define the research question first, clarify what is really being asked, split one big question into sub-questions, or rewrite a topic into 2-3 research directions. Do not use for simple fact lookup or article drafting. Default behavior: if the question still needs framing, produce a compact research brief and stop; if it is already researchable, emit a structured handoff payload for an outer agent to invoke deep-research."
---

# Pinpoint

Use this skill as a general research entrypoint.

Your job in phase 1 is not to answer the original question. Your job is to decide whether it already is researchable.

This skill has exactly two terminal states:

- `needs_framing`: the question is too broad, vague, or mixed and needs narrowing
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

## Readiness Criteria

Mark the question as `handoff_ready` only when all of the following are true:

- the research object or subject is clear
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

- `standard` is the default
- `quick` is for exploratory but still researchable requests
- `deep` is for high-stakes, decision-shaping, or explicitly rigorous requests
- `ultradeep` is only for maximum-comprehensiveness requests explicitly signaled by the user

## Phase 1 Workflow

Run these steps in order:

1. Preserve the user's original question.
2. Diagnose whether the question is `needs_framing` or `handoff_ready`.
3. Infer the likely research goal behind the question when the intent is clear.
4. If intent is ambiguous, present alternatives as assumptions rather than silently choosing one.
5. Define the minimum necessary boundaries:
   - object or subject
   - time range
   - geography or context
   - evaluation criteria
   - what is out of scope
6. If the question is too broad, rewrite it into 2-3 candidate research directions.
7. Recommend the version that is most researchable.
8. If the terminal state is `needs_framing`, produce a semi-structured research brief.
9. If the terminal state is `handoff_ready`, emit the structured handoff payload for `deep-research`.
10. Stop.

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
- `Terminal State`
- `Why It Needs Narrowing`
- `Recommended Research Version`
- `Boundaries`
- `Stop / Continue`

Add these sections when they materially help:

- `Problem Diagnosis`
- `Research Goal`
- `Key Terms And Boundaries`
- `Candidate Research Versions`
- `Research Brief`

### Output Shape: `handoff_ready`

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
  "next_action": "Invoke deep-research with the payload above"
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

## Research Brief

When you include a research brief, keep it short and practical. Summarize:

- the chosen research question
- the research goal
- the boundaries
- the key terms that matter for this round
- what is explicitly out of scope
- a one-line next research action after user confirmation

## Quality Bar

The goal is not to sound smart. The goal is to make the question researchable.

Prefer:

- narrower scope
- explicit boundaries
- testable wording
- direct research utility

Avoid:

- answering the full question too early
- keeping the wording broad but impressive
- over-defining irrelevant terms
- silently replacing the user's intent
- forcing one framing when multiple research directions are plausible

## Stop Rule

After phase 1, stop in one of two ways:

- if `needs_framing`, stop and wait for user confirmation before going further
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
