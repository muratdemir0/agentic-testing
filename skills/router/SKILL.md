# Router

## Purpose

Route testing requests to the correct skill: `acceptance-test-writer` for behavior-focused tests, `approval-test-writer` for output-focused tests. When a request involves both behavior and output, recommend both skills in sequence with a clear handoff point.

## When to Use

Use this skill when:

- You are unsure which test type applies to your scenario
- A request mentions both behavior and output and you need a sequencing recommendation
- You want a concise explanation of why one approach fits better than the other

## Routing Rules

### Route to `acceptance-test-writer`

The request mentions any of:

- behavior, flow, journey, user can, business rule
- login, checkout, onboarding, registration, authentication, access control
- end-to-end, browser, UI, page, form submission, navigation
- confirmation, redirect, error message, visible outcome
- payment success, payment failure, coupon, cart, subscription

### Route to `approval-test-writer`

The request mentions any of:

- output, diff, approved file, golden master, characterization
- JSON, HTML, PDF, XML, CSV, plain text
- receipt, invoice, report, document, email body
- template, serializer, renderer, generator
- snapshot, stable, noisy, scrub, approve all, diagnosability

### Conflict Resolution

If the request involves **both** behavior and output:

1. Use `acceptance-test-writer` first to cover the behavioral flow
2. Then use `approval-test-writer` to cover the generated output

This sequence reflects the natural dependency: you must first confirm that the system *did* the right thing before asserting exactly *what* it produced.

**Example:**
> "Test the checkout flow and make sure the receipt JSON is correct"

→ Step 1: `acceptance-test-writer` — test that the user can complete checkout  
→ Step 2: `approval-test-writer` — test that the receipt JSON matches the approved file

### Ambiguous Requests

If the request cannot be confidently classified, ask one clarifying question:

> "Are you testing **what the system does** (a behavior or user journey), or **what the system produces** (a generated output like JSON, HTML, or a document)?"

Do not ask multiple questions. One question resolves most ambiguous cases.

## Inputs

- A natural-language description of the testing need (anything from one sentence to a paragraph)

## Outputs

- The name of the recommended skill (`acceptance-test-writer` or `approval-test-writer`)
- A one-sentence explanation of why
- If both apply: a numbered sequence with a clear handoff point between the two skills
- If ambiguous: one targeted clarifying question

## Workflow

1. Read the request
2. Scan for trigger keywords (see routing rules above)
3. If only behavior keywords match → `acceptance-test-writer`
4. If only output keywords match → `approval-test-writer`
5. If both match → recommend both in sequence with explanation
6. If neither matches clearly → ask the one clarifying question

## Routing Table

| Request | Route |
|---|---|
| "Write a test for the checkout flow" | acceptance-test-writer |
| "Test that the receipt JSON is correct after purchase" | approval-test-writer |
| "Test that users can log in with invalid credentials" | acceptance-test-writer |
| "Approval test for the invoice HTML template" | approval-test-writer |
| "Test checkout and verify the receipt output is stable" | both, in sequence |
| "Snapshot test for the report generator" | approval-test-writer |
| "Test the coupon code application" | acceptance-test-writer |
| "Golden master test for the email template" | approval-test-writer |
| "Test the onboarding flow and verify the welcome email content" | both, in sequence |
| "Characterization test for our serializer" | approval-test-writer |
| "End-to-end test for payment failure handling" | acceptance-test-writer |

## Example System Prompt

```
You are a test routing assistant for a team that writes acceptance tests and approval tests.

Read the user's request and determine whether it describes:
- Behavior: what a user does, what the system does in response, a user journey, a business rule
- Output: what the system produces or generates — JSON, HTML, PDF, receipts, reports, serialized data

Route behavior requests to: acceptance-test-writer
Route output requests to: approval-test-writer

If both apply, recommend both in sequence: acceptance-test-writer first (to validate the behavior), then approval-test-writer (to validate the output produced by that behavior).

If unclear, ask exactly one question: "Are you testing what the system does, or what the system produces?"
```

## Preferred Response Format

1. Recommended skill(s) — stated first, clearly
2. One-sentence explanation of the routing decision
3. If both skills apply: numbered sequence with a one-line description of what each skill covers
4. If ambiguous: the clarifying question only — no speculation
