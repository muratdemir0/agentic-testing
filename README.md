# agentic-testing

> Agent skills for writing acceptance tests and approval tests — with clear routing between them.

## Why This Exists

Acceptance tests and approval tests are often confused. They are not the same thing.

| | Acceptance Tests | Approval Tests |
|---|---|---|
| **Focus** | Behavior | Output |
| **Question** | Does the system do the right thing? | Does the system produce the right output? |
| **Verified by** | Assertions on state and UI | Diff against an approved file |
| **Failure means** | A behavior changed | An output changed |
| **Typical tool** | Playwright, Cypress, RSpec | ApprovalTests, Jest snapshots, custom diff |
| **Best for** | Checkout flows, login, user journeys | Receipts, reports, JSON, HTML, PDF |

This confusion leads to real problems:

- Brittle tests that break on cosmetic output changes when an acceptance test was used instead
- Missing behavioral coverage because the team only diffed output
- Hard-to-diagnose failures because the wrong abstraction was applied

This repository provides three agent skills that enforce the distinction:

- **`acceptance-test-writer`** — Playwright-first, behavior-focused test writing
- **`approval-test-writer`** — Language-agnostic, output-focused test writing
- **`router`** — Routes your request to the right skill based on keywords

## Philosophy

**Behavior and output are different things.**

An acceptance test asks: *Did the user successfully check out?* It asserts that the confirmation page appeared, that the order was recorded, that the confirmation email was triggered.

An approval test asks: *Does the receipt look exactly like this?* It serializes the receipt, compares it to a golden master, and fails if anything changed.

You might use both in the same feature. The router handles this case explicitly.

**Approval tests are not just snapshots.**

Snapshot testing is a common pattern, but approval tests go further:

- They require *conscious review* before you approve output — not blind `approve all`
- They demand *determinism* — timestamps, UUIDs, and random values must be scrubbed or replaced before comparison
- They prioritize *diagnosability* — the diff must be readable and tell you exactly what changed
- The approved file is a first-class artifact checked into version control, not a generated side effect

**The approval-test-writer skill is language-agnostic.**

It adapts to the project's existing test framework. Whether you use Go, TypeScript, Python, Ruby, or Java, the skill generates approval tests in your language. If no language is specified, it chooses a sensible fallback and states the assumption clearly. See [`examples/go-approval-receipt.md`](examples/go-approval-receipt.md) for an example in Go and [`examples/playwright-checkout.md`](examples/playwright-checkout.md) for an acceptance test example.

## Skills

### acceptance-test-writer

Use this when you need to verify **behavior**: user journeys, business rules, visible outcomes.

**Best for:** checkout, login, onboarding, payment success/failure, coupon validation, form submission, access control, browser-based flows.

**Not for:** asserting the exact content of a serialized response, diffing a report against a golden master.

### approval-test-writer

Use this when you need to verify **output**: anything that gets serialized, rendered, or generated.

**Best for:** receipts, invoices, JSON responses, HTML templates, PDF rendering, email bodies, report output, serializers.

**Not for:** testing whether a user can complete a checkout flow.

### router

Use this when you are not sure which skill to use. Describe your testing need in plain language and the router will direct you to the right skill, or recommend both in sequence if the request involves behavior and output.

## Example Requests

```
"Write a Playwright test for the checkout flow including coupon code application"
→ acceptance-test-writer

"Write an approval test for the receipt JSON generated after a successful order"
→ approval-test-writer

"I need to test my invoice generator — it produces a PDF and I want to catch unintended changes"
→ approval-test-writer

"Test that users can log in, add items to cart, and complete purchase"
→ acceptance-test-writer

"Test the checkout flow and also verify the receipt JSON output is stable"
→ router → acceptance-test-writer first, then approval-test-writer
```

## Target Audience

- Engineering teams writing end-to-end or integration tests
- Developers using AI assistants for test generation
- Teams adopting snapshot or golden-master testing patterns
- Anyone who has mixed up acceptance and approval testing before

## skills.sh Compatibility

Each skill is a standard `SKILL.md` file compatible with [skills.sh](https://skills.sh). Install individually:

```sh
npx skills add muratdemir0/agentic-testing@acceptance-test-writer
npx skills add muratdemir0/agentic-testing@approval-test-writer
npx skills add muratdemir0/agentic-testing@router
```

## Future Scope

- `mutation-test-advisor` — helps identify undertested code paths using mutation testing concepts
- `test-reviewer` — reviews existing tests for quality issues (brittle selectors, missing failure paths, over-mocking)
- `coverage-advisor` — summarizes coverage gaps and suggests which skill to apply
