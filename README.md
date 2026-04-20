# Agent Skills

A collection of skills for AI coding agents by Charlie Tango. Skills are packaged instructions and references that extend agent capabilities.

Skills follow the [Agent Skills](https://agentskills.io/) format.

## Available Skills

### react-effect-audit

Enforce strict scrutiny around `useEffect` in React and Next.js code. Prefer non-Effect patterns from React's "You Might Not Need an Effect" guidance.

**Use when:**
- Reviewing React or Next.js components
- Refactoring hooks and client-side state logic
- Generating new UI code where `useEffect` might be introduced

**Focus areas:**
- Remove unnecessary Effects
- Justify any remaining Effect with an explicit external system
- Prefer render derivation, event handlers, keys, lifted state, refs, and `useSyncExternalStore`
- Treat `useMemo` as optional and justified, not default

## Installation

```bash
npx skills add https://github.com/charlie-tango/agent-skills
```

## Usage

Skills are available after installation. The agent should apply them when tasks match the skill intent.

**Examples:**
```text
Review this React component for unnecessary useEffect usage
```
```text
Refactor this Next.js client component to avoid effect-based derived state
```

## Skill Structure

Each skill contains:
- `SKILL.md` - Instructions for the agent
- `agents/` - Integration metadata (optional)
- `references/` - Supporting documentation (optional)

## Contributing

1. Add or update a skill folder under `skills/`
2. Keep instructions concrete, testable, and scoped
3. Open a PR with a short rationale and example use case
