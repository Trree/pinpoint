# Pinpoint

`Pinpoint / 定点` is an extensible research entry router.

It does not only clarify vague questions. It first analyzes the request through a compact framing pipeline, then decides the best next route:

- `needs-confirmation`
- `deep-research`
- `light-answer`
- `skill-route`

The original clarification behavior is preserved as the `needs-confirmation` route.

## Compatibility

- Claude Code: yes
- Gemini CLI: yes
- Codex-style environments: yes
- Chat apps or web UIs without skill loading: not directly; use the `SKILL.md` content as a manual system prompt instead

## What Is Portable

The portable core of this skill is:

- `SKILL.md`
- `references/`

The file `agents/openai.yaml` is only an environment-specific adapter for the current workspace.

## Install

### Claude Code

Install `pinpoint` into your Claude skills directory:

```bash
ln -s /absolute/path/to/pinpoint ~/.claude/skills/pinpoint
```

Or copy it:

```bash
cp -R /absolute/path/to/pinpoint ~/.claude/skills/pinpoint
```

### Gemini CLI

Place the `pinpoint` directory under your Gemini CLI skills directory or configured skills root, then activate it by name:

```text
activate_skill pinpoint
```

## Structure

```text
pinpoint/
├── SKILL.md
├── README.md
├── agents/
│   └── openai.yaml
└── references/
    ├── output-examples.md
    ├── phase-2-framework.md
    ├── phase-3-evidence.md
    ├── phase-4-counterarguments.md
    ├── phase-5-scenarios.md
    └── phase-6-synthesis.md
```

## Core Flow

Pinpoint runs this sequence:

1. `Trend Scan`
2. `Anti-Consensus Scan`
3. `Perspective Expansion`
4. `Consolidated Brief`
5. `Routing Decision`

The first three stages are framing inputs. They must always converge into one `Consolidated Brief` before routing.

## Route Targets

### `needs-confirmation`

Use when the request still needs user confirmation before downstream execution.

This is the preserved form of the original Pinpoint clarification workflow.

### `deep-research`

Use when the request is converged and substantial enough for multi-source synthesis, verification, or report-style analysis.

### `light-answer`

Use when the request is clear but not substantial enough for `deep-research`.

Pinpoint does not answer directly. It emits a lightweight-answer payload for an outer agent or lighter downstream workflow.

### `skill-route`

Use when the request is clear and a specific skill is a better next step than `deep-research` or `light-answer`.

## Analysis Contract

Pinpoint conceptually produces an `analysis_block` with:

- `original_question`
- `trend_signals`
- `anti_consensus_angles`
- `perspective_lenses`
- `consolidated_brief`

`consolidated_brief` is the authoritative basis for all routing. Pinpoint must not route directly from raw insight lists.

## Routing Contract

Pinpoint conceptually produces:

- `routing_decision`
- `route_payload`

The canonical internal route shell is:

```json
{
  "route_version": "2.0",
  "target": "deep-research | light-answer | needs-confirmation | skill-route",
  "payload": {}
}
```

Outer agents should:

- read `target`
- parse only the payload for that target
- avoid assuming one target's payload shape applies to another
- treat `needs_user_confirmation` as true only for the `needs-confirmation` route

## Deep Research Compatibility

When `target = deep-research`, Pinpoint must preserve the current legacy top-level handoff JSON shape as the user-facing output.

Canonical constants that remain fixed:

- `handoff_version: "1.1"`
- `terminal_state: "handoff_ready"`
- `target_skill: "deep-research"`
- `next_action: "Invoke deep-research with this payload"`

This route remains compatibility-first.

## Output Boundaries

### `deep-research`

- optional one-line lead-in
- then legacy top-level JSON only

### `needs-confirmation`

- readable clarification brief
- then exactly one fenced `json` block containing the canonical route shell

### `light-answer`

- short routing summary
- then exactly one fenced `json` block containing the canonical route shell

### `skill-route`

- short routing summary
- then exactly one fenced `json` block containing the canonical route shell

## Notes

- Pinpoint does not execute downstream skills itself.
- Pinpoint does not gather deep-research evidence itself.
- Pinpoint does not generate full content drafts.
- Pinpoint's job is to improve the next decision, not to finish the whole workflow.
