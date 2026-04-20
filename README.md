# Charlie Tango Agent Skills

A curated repository of reusable agent skills used by Charlie Tango.

Each skill is self-contained and designed to be:
- Easy to discover
- Safe to apply in real projects
- Practical for day-to-day engineering work

## What Is A Skill?

A skill is a folder with a `SKILL.md` that defines:
- When the skill should be used
- The workflow and decision rules the agent should follow
- Any extra references or agent metadata needed by integrations

## Repository Structure

```text
skills/
  <skill-name>/
    SKILL.md                 # Core instructions and usage criteria
    agents/
      openai.yaml            # Optional integration metadata
    references/              # Optional deep-dive docs
README.md
```

## Current Skills

### `react-effect-audit`
Enforces strict scrutiny of `useEffect` usage in React/Next.js code, based on React's "You Might Not Need an Effect" guidance.

Primary behaviors:
- Require explicit justification for each kept or added effect
- Prefer non-effect alternatives (derive in render, event handlers, key resets, lifted state)
- Treat `useMemo` as optional optimization, not default control flow

Location: `skills/react-effect-audit/`

## Usage

If your agent runtime supports local skills, point it at this repository (or copy selected skill folders into your runtime skill directory), then invoke skills by name when the task matches.

At minimum, each skill must include:
1. Front matter (`name`, `description`) in `SKILL.md`
2. Clear trigger conditions
3. Step-by-step instructions the agent can execute consistently

## Adding A New Skill

1. Create a new folder: `skills/<your-skill-name>/`
2. Add `SKILL.md` with front matter and actionable instructions
3. Add `agents/openai.yaml` if integration metadata is needed
4. Add `references/` docs only when extra depth is useful
5. Keep instructions concrete and testable on real tasks

### `SKILL.md` starter template

```md
---
name: your-skill-name
description: >-
  One concise paragraph describing when and why to use this skill.
---

# Skill Title

## Purpose
State what the skill optimizes for.

## When to use
- Trigger condition 1
- Trigger condition 2

## Workflow
1. Step one
2. Step two
3. Step three

## Output expectations
- Expected response format or quality bar
```

## Authoring Principles

- Be explicit about decision criteria
- Prefer deterministic, low-ambiguity instructions
- Include anti-patterns to avoid
- Optimize for practical code outcomes, not generic advice

## Contributing

1. Open a PR with the new or updated skill folder
2. Include a short rationale and example use case
3. Keep changes scoped to the skill being improved
