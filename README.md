# Pinpoint

`Pinpoint / 定点` is a research-framing skill. It turns broad, vague, or mixed questions into a clearer and more researchable version before deeper analysis starts.

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

Phase 1 only:

- preserve the original question
- diagnose whether it is already researchable
- narrow scope and boundaries where needed
- propose 2-3 candidate research directions when necessary
- recommend one version
- produce a compact research brief
- stop for confirmation

This skill is intentionally front-loaded. Its job is to frame the research well before evidence gathering starts.
