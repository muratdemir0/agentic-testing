# Acceptance Test Writer

## Purpose

Write acceptance tests that validate user behavior, business rules, and visible outcomes. This skill is Playwright-first but applies the same behavioral thinking to any end-to-end or integration test framework.

## When to Use

Use this skill when the testing need involves:

- A user completing a journey (checkout, login, onboarding, registration)
- A business rule being enforced (coupon validation, payment limits, access control)
- A visible outcome being asserted (confirmation page, error message, redirect, email triggered)
- Browser-based flows with UI interactions
- End-to-end tests that cross service boundaries

**Trigger keywords:** behavior, flow, journey, login, checkout, user can, business rule, end-to-end, onboarding, payment, form submission, redirect, confirmation, access, navigation

## When Not to Use

Do not use this skill when:

- The primary concern is the exact content of a serialized output (JSON, HTML, PDF, text)
- You are testing a pure function's return value
- You need to diff a report or document against a golden master
- The test subject has no user-facing behavior

Use `approval-test-writer` instead for output-focused testing.

## Inputs

- Description of the user journey or business scenario
- URL, page, or component under test
- User roles and preconditions (e.g., authenticated user, cart with items, seeded product catalog)
- Expected outcomes (e.g., confirmation page appears, order is persisted, email is triggered)
- Failure scenarios to cover (optional but strongly recommended)

## Outputs

- A Playwright test file in TypeScript (default) or the project's preferred language
- Test setup and teardown hooks where appropriate
- Page Object Model structure for complex multi-page flows
- Comments explaining the intent of each test block
- Guidance on selectors used and why

## Principles

1. **Test behavior, not implementation.** Assert what the user sees and what the system does — not how it does it internally.
2. **Use semantic selectors.** Prefer `getByRole`, `getByLabel`, `getByText` over CSS selectors, `data-testid` shortcuts, or nth-child hacks.
3. **Assert visible outcomes.** Every assertion should correspond to something the user or a stakeholder cares about.
4. **Cover the happy path and key failure paths.** A checkout test must cover successful purchase AND payment failure. The failure path is not an edge case.
5. **Avoid brittle selectors.** Do not use absolute XPaths, layout-dependent selectors, or selectors tied to styling classes.
6. **Keep tests independent.** Each test must be runnable in isolation without depending on another test's side effects.
7. **Name tests as sentences.** `test('user can complete checkout with valid coupon')` not `test('checkout 3')`.

## Workflow

1. Understand the user journey being tested — write it in plain English first if helpful
2. Identify preconditions: authentication state, seeded data, feature flags
3. Write the happy path test
4. Add at least one failure path test (invalid input, declined payment, unauthorized access)
5. Extract reusable steps into a Page Object Model if multiple pages or repeated interactions appear
6. Assert on visible, meaningful outcomes — not internal state or implementation details
7. Review each selector for brittleness before finalizing

## Writing Rules

- Use `page.getByRole()`, `page.getByLabel()`, `page.getByText()` as the primary selector strategy
- Use `expect(locator).toBeVisible()`, `expect(locator).toHaveText()`, `expect(page).toHaveURL()` for assertions
- Never assert on CSS class names or arbitrary element IDs
- Never import application modules to inspect internal state from a test
- Wrap multi-step flows in `test.step()` blocks for readability and better failure reporting
- Group related scenarios under `test.describe`
- Use `test.beforeEach` for common setup (authentication, navigation to starting page)
- For authenticated flows, prefer stored auth state over logging in on every test

## Example Requests

```
"Write a Playwright test for the checkout flow with coupon application and payment confirmation"
"Write an acceptance test for the login page covering invalid credentials and account lockout"
"Test the onboarding flow for new users including email verification step"
"Write end-to-end tests for the subscription upgrade flow"
"Test that an unauthenticated user is redirected to login when accessing account settings"
```

## Example System Prompt

```
You are an expert Playwright test writer. Write behavior-focused acceptance tests.

Use semantic selectors: getByRole, getByLabel, getByText. Never use CSS classes or data-testid unless they carry semantic meaning.

Assert visible, meaningful outcomes — what the user sees, not what the database stores.

Cover the happy path and at least one important failure scenario.

Structure tests with test.describe and test.step. Name every test as a complete sentence describing user behavior.

Use Page Object Models for multi-page or multi-step flows.
```

## Preferred Response Format

1. Brief test strategy summary (2–3 sentences): what is being tested, what preconditions exist, what the happy and failure paths cover
2. Full Playwright test file in TypeScript
3. Page Object Model files if applicable
4. Notes on selector choices and assertion rationale
5. Suggestions for additional failure paths not yet covered
