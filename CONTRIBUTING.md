# Contributing to agentic-testing

## What This Repository Is

A collection of agent skills for writing acceptance tests and approval tests, with a router skill to choose between them. The skills are Markdown files designed to guide AI coding assistants, work with skills.sh, or be embedded in system prompts directly.

## Core Constraint

**The distinction between acceptance tests and approval tests is the reason this repository exists.** Every contribution must preserve or sharpen it. Do not add skills or examples that blur this distinction.

## What Good Contributions Look Like

- Skills that are practical, specific, and demonstrably useful to real engineering teams
- Examples that illustrate a concrete scenario with realistic code
- Improvements to routing accuracy with clear justification
- Corrections to normalization guidance in `approval-test-writer`
- New examples in languages not yet covered (the skill is language-agnostic — examples should reflect that)

## What to Avoid

- Adding language-specific bias to `approval-test-writer` — it must remain language-agnostic
- Merging behavioral and output testing concerns into a single skill
- Over-engineering skill files with unnecessary sections or abstract guidance
- Adding files that are not skills, examples, or documentation

## Skill File Structure

Each skill lives at:

```
skills/<skill-name>/SKILL.md
```

Every `SKILL.md` must include these sections in order:

- Skill name and purpose
- When to use
- When not to use
- Inputs
- Outputs
- Principles
- Workflow
- Writing rules
- Example requests
- Example system prompt
- Preferred response format

Do not add sections not in this list without discussion.

## Adding a New Skill

1. Create a directory under `skills/` with a lowercase hyphenated name
2. Write `SKILL.md` following the structure above
3. Add at least one example under `examples/` if the skill has non-obvious usage
4. Add the skill to the skills table in `README.md`
5. Open a pull request with a one-paragraph description of the skill's purpose and target user

## Adding an Example

Examples live in `examples/`. File names follow the pattern `<framework-or-language>-<scenario>.md`.

Each example must:

- State which skill generated it at the top
- Be realistic and production-quality — not a toy
- Include notes explaining key design decisions (selector choices, normalization strategy, etc.)
- Cross-reference the complementary example where relevant (e.g., the checkout acceptance test links to the receipt approval test)

## Modifying an Existing Skill

- Explain what changed and why in the pull request description
- If you are changing routing keywords in `router/SKILL.md`, include at least two example requests that demonstrate the routing improvement
- If you are changing the language policy in `approval-test-writer`, ensure the change keeps the skill genuinely language-agnostic

## Pull Request Guidelines

- One skill, one example, or one focused improvement per PR
- No generated files, no build artifacts, no lock files
- Keep the tone practical and direct — this is an engineering resource, not documentation for its own sake

## Questions

Open an issue. Label it `question`. Be specific about which skill or example the question concerns.
