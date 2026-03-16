---
name: pinpoint
description: "Turn broad, vague, or mixed questions into clear research problems before deeper analysis. Use when the user wants to research, analyze, compare, evaluate, or think through a topic but the question still needs framing, narrowing, decomposition, or boundary-setting. Also use when the user explicitly asks to define the research question first, clarify what is really being asked, split one big question into sub-questions, or rewrite a topic into 2-3 research directions. Do not use for simple fact lookup, article drafting, or already-scoped retrieval tasks. Default behavior: produce a researchable version of the question, a compact research brief, and then stop for confirmation."
---

# Pinpoint

Use this skill as a general research entrypoint.

Your job in phase 1 is not to answer the original question. Your job is to make the question researchable.

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

If the request is already researchable:

- say so briefly
- do not force candidate rewrites
- either stop with a readiness judgment or hand off to the next research workflow

## Phase 1 Workflow

Run these steps in order:

1. Preserve the user's original question.
2. Diagnose why the question is not yet ready for direct research, or state that it is already researchable.
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
8. Produce a semi-structured research brief.
9. Stop and wait for user confirmation.

## Assumption Policy

- If intent is clear, infer the likely research goal and label it as inferred.
- If intent is ambiguous, present alternatives explicitly as assumptions.
- Ask a clarification question only when the ambiguity would materially change the research direction.
- Do not recommend a rewrite unless the original question is genuinely too broad, too vague, or mixed across multiple problems.

## Output Shape

Use a semi-structured format with this compact required backbone:

- `Original Question`
- `Why It Needs Narrowing` or `Already Researchable`
- `Recommended Research Version`
- `Boundaries`
- `Stop / Continue`

Add these sections when they materially help:

- `Problem Diagnosis`
- `Research Goal`
- `Key Terms And Boundaries`
- `Candidate Research Versions`
- `Research Brief`

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

After phase 1, stop and wait for user confirmation before moving on.

Do not automatically proceed into:

- evidence gathering
- framework building
- counterarguments
- scenario analysis
- recursive follow-up research

Stop boundary:

- allow at most a one-line next step
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
