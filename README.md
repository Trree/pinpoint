# Pinpoint

`Pinpoint / 定点` is a research-framing skill. It acts as a research entry router:

- if the question is broad, vague, or mixed, it first expands useful perspectives and then converges into a clearer research brief
- if the question is already researchable and substantial enough for `deep-research`, it emits a structured handoff payload for an outer agent to invoke `deep-research`

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

Then invoke it in Claude Code with natural language, for example:

```text
Use pinpoint to turn this topic into a researchable question: AI agents in healthcare
```

### Gemini CLI

Place the `pinpoint` directory under your Gemini CLI skills directory or configured skills root, then activate it by name:

```text
activate_skill pinpoint
```

Example prompt after activation:

```text
Turn this topic into 2-3 researchable directions and a short brief: AI agents in healthcare
```

Notes:

- `Pinpoint` does not depend on subagents.
- Gemini CLI's lack of Claude-style `Task` support does not block this skill.
- Tool names referenced by shared skill infrastructure may need Gemini CLI equivalents such as `activate_skill`.

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

## Usage Contract

Core flow only:

- preserve the original question
- run trend capture
- run anti-consensus exploration
- run multi-perspective generation
- converge into one consolidated research brief
- diagnose whether it needs framing or is ready for handoff
- only mark handoff-ready when the task also fits `deep-research` rather than a simple lookup
- narrow scope and boundaries where needed
- propose 2-3 candidate research directions only when convergence is still weak
- produce a compact research brief when framing is still needed
- emit a structured `deep-research` handoff payload when the question is already researchable
- stop

This skill is intentionally front-loaded. Its job is to improve question quality before deeper analysis starts. It does not gather evidence or execute `deep-research` itself.

## Perspective Expansion

Before narrowing, `Pinpoint` now runs three framing stages:

- `Trend Scan`: capture high-value directional signals
- `Anti-Consensus Scan`: surface underexplored or weakly challenged angles
- `Perspective Expansion`: generate 3 useful framing lenses before convergence

These stages are inputs to one `Consolidated Research Brief`. `Pinpoint` must not stop at a raw insight list or hand off directly from divergent lens output.

## Handoff Contract

When `Pinpoint` determines the question is already researchable and suitable for `deep-research`, it should end in `handoff_ready` and emit a stable JSON payload for an outer agent. The outer agent is responsible for invoking `deep-research`.

Core payload fields:

- `handoff_version`
- `terminal_state`
- `target_skill`
- `research_question`
- `recommended_mode`
- `research_goal`
- `boundaries`
- `assumptions`
- `key_terms`
- `success_criteria`
- `next_action`

This keeps `Pinpoint` focused on framing and routing, while `deep-research` remains responsible for retrieval, verification, synthesis, and report generation.

Outer-agent expectations:

- validate the payload before dispatch
- treat inferred time ranges as normalized absolute values in the payload
- honor deterministic mode selection: `standard` by default, `quick` for exploratory asks, `deep` for high-stakes asks, `ultradeep` only when explicitly requested
- if validation fails or `deep-research` is unavailable, do not auto-dispatch
- if `deep-research` rejects the task as too lightweight, fall back to a lighter workflow instead of looping back into `Pinpoint`
- if `deep-research` fails operationally, surface the payload plus the error and stop

When `Pinpoint` ends in `needs_framing`, the output should be a human-readable brief with:

- `Trend Signals`
- `Anti-Consensus Angles`
- `Perspective Lenses`
- `Consolidated Research Brief`
- `Researchability Check`
- `Recommended Framing`
- `Next User Action`
