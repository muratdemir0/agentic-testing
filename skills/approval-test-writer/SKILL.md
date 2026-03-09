# Approval Test Writer

## Purpose

Write approval tests that validate generated output by comparing it against a known-good approved file. This skill is **language-agnostic** and adapts to the project's existing test framework and language.

Approval testing is also known as: golden master testing, characterization testing, snapshot testing (with stronger review requirements). The core idea is always the same: serialize the output, compare it to an approved version, fail if it differs.

## When to Use

Use this skill when the testing need involves:

- A serializer, renderer, or generator that produces output
- JSON responses that should remain stable across changes
- HTML templates, PDF output, or email bodies
- Reports, receipts, invoices, or documents
- Any output where "did this change?" is the meaningful question

**Trigger keywords:** output, diff, approved file, golden master, characterization, JSON, receipt, report, HTML, PDF, XML, CSV, template, serializer, renderer, generator, snapshot, stable, noisy, scrub, approve all, diagnosability

## When Not to Use

Do not use this skill when:

- You are testing whether a user can complete a flow or journey
- The test subject is a behavior, not an output
- You need to assert on application state or database records
- You have no serializable, stable output to approve against

Use `acceptance-test-writer` instead for behavior-focused testing.

## Language Policy

**This skill is language-agnostic.**

It adapts to the project's existing language and test framework. The approval testing pattern is consistent across languages — only the syntax and libraries change.

**If you specify a language**, the skill generates tests in that language using idiomatic patterns and the most appropriate approval testing library for that ecosystem:

| Language | Common Approach |
|---|---|
| TypeScript / JavaScript | Jest with custom approved file helper, or `jest-snapshot` with careful review |
| Go | `go-approval-tests` library |
| Python | `pytest-approvaltests` or custom file comparison |
| Ruby | `approvals` gem |
| Java | `ApprovalTests.Java` |
| C# | `ApprovalTests` (the original library) |
| Any | Custom: serialize → write received file → diff against approved file |

**If you do not specify a language**, the skill will:

1. Infer the language from context (file names, imports, existing test examples provided)
2. If inference is not possible, default to **TypeScript with Jest**
3. Clearly state the assumed language and framework at the top of the response

## Inputs

- Description of the output being tested: what generates it, what format it produces
- Language and test framework (if known — highly recommended)
- Example output or schema (optional but accelerates the workflow)
- Known unstable fields: timestamps, UUIDs, random IDs, auto-incremented values, environment-specific paths
- Preferred approved file format: inline string, `.approved.txt`, `.approved.json`, etc.

## Outputs

- A test file in the project's language with approval test structure
- Scrubbing/normalization helper functions for each unstable field
- A sample `.approved` file (or instructions for generating one on first run)
- Comments explaining each normalization decision and why it is necessary
- First-run approval instructions and re-approval workflow

## Principles

1. **Output is a contract.** The approved file is not a side effect — it is a specification of what the output should look like. Treat it as such.
2. **Determinism is required.** Any field that changes between runs must be scrubbed before comparison. This includes timestamps, UUIDs, random IDs, auto-incremented values, and environment-specific paths. Non-deterministic output produces noisy, unreliable tests.
3. **Diffs must be readable.** Structure output so that the diff is meaningful. JSON should be pretty-printed and consistently sorted. HTML should be indented. Avoid minified or compressed formats.
4. **Conscious approval.** The first time you run an approval test, review the output carefully before approving it. Do not use `approve all` without reading. Each approved file is a deliberate assertion.
5. **Diagnosability first.** When a test fails, the diff should tell you exactly what changed and whether the change is intentional. Good scrubbing eliminates noise so that real changes stand out.
6. **Noise reduction through scrubbing.** Scrubbing is not cheating — it is how you make approval tests sustainable long-term. Scrub what is inherently unstable; assert on everything else.

## Workflow

1. Identify the output: what is being generated and in what format
2. Identify unstable fields: timestamps, UUIDs, ordering, environment values, auto-incremented IDs
3. Write a normalization/scrubbing function for each unstable field
4. Generate the output in the test and run it once to produce the received file
5. Review the received output carefully — read it as a specification
6. Approve it by saving as the `.approved` file (or using the library's approval command)
7. Future runs diff against the approved file automatically
8. When legitimate changes occur (e.g., new field added), review the diff and re-approve deliberately

## Normalization Techniques

### Timestamps
Replace with a fixed value at construction time, or scrub with a regex after serialization:
- Inject: `createdAt: time.Date(2024, 1, 1, 0, 0, 0, 0, time.UTC)`
- Scrub: replace `\d{4}-\d{2}-\d{2}T\d{2}:\d{2}:\d{2}Z` with `<timestamp>`

### UUIDs and Random IDs
Replace with deterministic values at construction time, or scrub to a stable placeholder:
- Inject: `id: "00000000-0000-0000-0000-000000000001"`
- Scrub: replace UUID pattern with `<id>`

### Ordering
Sort collections before serializing when order is not semantically meaningful. Document the sort key.

### Environment-Specific Values
Replace hostnames, file system paths, environment names, and version numbers with placeholders or fixed values.

### Noisy Fields
If a field changes frequently and carries no meaningful signal for this test, omit it from the serialized output entirely. Add a comment explaining the omission.

## Writing Rules

- Always normalize unstable fields before comparison — never commit to an approved file that contains live timestamps or random IDs
- Never approve output without reading it
- Store approved files in version control alongside the tests — they are specifications
- Name approved files clearly and consistently: `receipt_test.approved.json`, not `snap1.txt`
- Write one approval test per logical output unit, not one per field
- Prefer pretty-printed, human-readable output for better diffs
- Add inline comments explaining why each normalization function exists
- Do not use `approve all` scripts in CI — approvals must be a conscious local action

## Example Requests

```
"Write an approval test for the receipt JSON returned after a successful order"
"Write an approval test for the HTML invoice template rendered by our billing service"
"I have a PDF generator — write tests that catch unintended changes to its output"
"Write a characterization test for our report serializer in Python"
"Write a golden master test for the email template rendered by our notification service"
```

## Example System Prompt

```
You are an expert at writing approval tests for generated output.

Identify unstable fields — timestamps, UUIDs, ordering, environment-specific values — and normalize them before comparison.

Generate tests in the project's existing language and test framework. If the language is not specified, use TypeScript with Jest and clearly state this assumption at the top of your response.

Store approved output as a separate file in version control. Name it clearly.

Prioritize diff readability and diagnosability. Scrub noise so that real changes are immediately visible.

Explain every normalization decision with an inline comment.
```

## Preferred Response Format

1. Summary of the output being tested and the normalization strategy (3–5 sentences), including the assumed language and framework if it was inferred
2. Scrubbing/normalization helper functions with comments
3. Full test file
4. Sample `.approved` file content
5. First-run approval instructions and re-approval workflow
